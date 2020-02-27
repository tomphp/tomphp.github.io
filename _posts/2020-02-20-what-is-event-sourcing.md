---
title: What is Event Sourcing?
description: In this article, I want to give a basic introduction to what Event Sourcing is, and share some thoughts from the community about deciding if it's right for your project.
featured_image: '/images/what-is-event-sourcing/featured-image.jpg'
---

In this article, I want to give a basic introduction to what Event Sourcing is,
and share some thoughts from the community about deciding if it's right for your
project.

I've had a keen interest in event sourcing for several years, however, in the
last two years I was not actively following the community. In that time I've
seen some posts and articles popping up on social media with tales of event
sourcing, but not in the way that I understood it. Often it would seem that I
was reading about Event Driven Architectures with a Kafka log stored somewhere.

I felt like I had fallen behind the times and that maybe Event Sourcing had
evolved. When [DDD Europe 2020](https://dddeurope.com/) announced that they were
going to hold a one day, pre-conference conference on Event Sourcing, I jumped
at the chance to get back up to speed. After attending this event (and the
fabulous DDD Europe main conference) I was happy to learn that Event Sourcing is
still what I knew it to be. More importantly, I also learned some valuable
lessons from the advice and experience shared in the talks.

## What Event Sourcing is

In the opening keynote at the event sourcing conference,
[Udi Dahan](https://twitter.com/UdiDahan) said:

> Event sourcing refers to a collection of patterns based on persisting the full
> history of a domain as a sequence of "events", rather than persisting just the
> current state.

### Storing Events

With event sourcing, we store a record of any event that changes the state,
rather than the resulting state after the event occurred. Events are stored as
immutable pieces of data that record what event happened, along with any data
that is relevant to that event. Events are immutable because they represent
something that has already happened (you cannot change the past). As such,
they are named as actions in the past tense, e.g. _OrderPlaced_, _PaymentTaken_,
_OrderShipped_.

Events are stored in streams, with a stream of events representing the entire
history of a particular entity (or _Aggregate_ in DDD terminology). These event
streams are the source of truth for the state of the aggregate.

Once actions have been performed on the aggregate, we need to be able to get the
events out to persist them in the event store.

As an example, I'm going to use an Order aggregate from an e-commerce system.
You can **create** an order, **add items** to it, and then **place** it. Below
you can see the public methods which perform these actions. In addition, there
is also retrieveNewEvents()which will return all new events that have been
generated since the object instance was constructed. These events can then be
persisted to the event store.

```java
public class Order {
    public static Order create(OrderID orderID) {
        // Create a new Order (generates OrderCreated)
    }

    public void addItem(ItemCode item) {
        // Add an item to the order (generates ItemAdded)
    }

    public void place() {
        // Place the order (generates OrderPlaced)
    }

    public List<OrderEvent> retrieveNewEvents() {
        // Return all new events have have been generated
    }

    // ...
}
```

### Replaying an Event Stream
To load an event sourced aggregate's state into memory, the stored events need
to be replayed in order. To be able to do this, we need need to have another
constructors for the aggregate which takes the stream of events

```java
public class Order {
    public static Order fromEventStream(ArrayList<OrderEvent> {
        // Create new Order and replay events to get the state
    }

    // ...
}
```

We now have two constructors:
* `create()` creates a brand new order and adds an initial OrderCreated event to
    the list of new events.
* `fromEventStream()` builds the current state from a stream of events. It also
    initialises the list of new events to an empty list.

Let's now take a look at a simplistic way that we might implement this. It is
possible to refactor this code into a better state, but I want to keep it as
simple as possible to highlight the key concepts.

To start with, let's look at the state in the `Order` class:

```java
private ArrayList<OrderEvent> newEvents = new ArrayList<>();

private OrderID id;
private HashMap<ItemCode, Quantity> lines = new HashMap<>();
private boolean placed = false;
```

`id`, `lines` and `placed` are the current state of the aggregate, while
`newEvents` is used to keep a track of events as they occur.

Let's now see that how the `create()` constructor works:

```java
public static Order create(OrderID orderID) {
    var orderCreated = new OrderCreated(orderID);
    var events = Collections.singletonList(orderCreated);
    var order = Order.fromEventStream(new ArrayList<>(events));
    order.newEvents.add(orderCreated);
    return order;
}
```

The code above creates a new stream, containing a single OrderCreated event, and
then creates the `Order` from that stream. This shows that actually
`fromEventStream()` is the important constructor. The only other thing that
`create()` does is save the event to `newEvents`.

Now let's look at `fromEventStream()`:

```java
public static Order fromEventStream(ArrayList<OrderEvent> events) {
    return new Order(events);
}

private Order(ArrayList<OrderEvent> events) {
    if (events.size() == 0) {
        throw new EmptyEventStream();
    }

    if (!(events.get(0) instanceof OrderCreated)) {
        throw new InvalidEvent(
                "The first event must be OrderCreated"
        );
    }

    for (OrderEvent event : events) {
        applyEvent(event);
    }
}
```

We see that `fromEventStream()` just delegates through to a private constructor.
The private constructor does a few checks on the event stream and then iterates
over it, calling `applyEvent()` with each event.

`applyEvent()` is a private method that takes all event types and mutates the
state accordingly. Here I have implemented it with an overloaded method and a
nasty switch statement to check the event type. There are better ways to do this
but it keeps the example simple:

```java
private void applyEvent(OrderEvent event) {
    if (event instanceof OrderCreated) {
        applyEvent((OrderCreated) event);
    } else if (event instanceof ItemAdded) {
        applyEvent((ItemAdded) event);
    } else if (event instanceof OrderPlaced) {
        applyEvent((OrderPlaced) event);
    }
}

private void applyEvent(OrderCreated event) {
    id = event.id;
}

private void applyEvent(ItemAdded event) {
    if (placed) {
        throw new OrderHasAlreadyBeenPlaced(id);
    }

    var quantity = lines.containsKey(event.itemCode)
            ? lines.get(event.itemCode).increment()
            : Quantity.of(1);

    lines.put(event.itemCode, quantity);
}

private void applyEvent(OrderPlaced event) {
    if (placed) {
        throw new OrderHasAlreadyBeenPlaced(id);
    }

    if (lines.isEmpty()) {
        throw new OrderHasNoItems(id);
    }

    placed = true;
}
```

We have now seen how our event sourced aggregate's state can be created from a
stream of events. The final piece of the puzzle is how we create new events.
With all the internal event driven infrastructure in place, this is very easy:

```java
public void addItem(ItemCode item) {
    OrderEvent event = new ItemAdded(item);
    applyEvent(event);
    newEvents.add(event);
}

public void place() {
    OrderEvent event = new OrderPlaced();
    applyEvent(event);
    newEvents.add(event);
}
```

Now we have the complete example, I hope that it's clear what event sourcing is.
This example has been highly trivialised though, the core concepts are clear but
event sourcing brings a lot of new complexity; for example:

* How do you deal with conflicts when you try and save new events but someone
    else has saved some new ones before you?
* How do you version events? Since they are immutable, the old versions still
    need to be understood by the system.
* What about performance when you have large event streams?
* How do you delete sensitive data from the history (e.g. comply with GDPR)?

The answers to these questions are beyond this scope of this article, but it's
important to know that the simple concept of event sourcing brings a lot of new
challenges.

You can find the complete code for the example
[here](https://github.com/tomphp/event-sourcing-example).

## When to use Event Sourcing

Now that we know what event sourcing is, when should we use it?

Interestingly, a fairly consistent message at the talks in the conference was a
strong warning against event sourcing all the things. Once you start thinking in
domain events it seems like everything should be event sourced because it maps
to the way we experience reality. However, the complexities of implementing this
powerful technique make it a trade-off that should be carefully considered.

First and foremost, you should consider what event sourcing gives you:
* A full audit log (also useful for understanding and debugging issues)
* The ability to restore the system to any point in history
* The ability to gather information retrospectively
* Decoupling the state from what happened

These all sound great, but the question is - _does the value event sourcing
brings to your solution outweigh the challenges to implement it?_

In addition to this, Udi also gave some guidance on where it's appropriate to
use event sourcing in his keynote. Firstly he defined event sourcing as _"a
collection of patterns based on persisting the full history of a **domain**"_.
He then talked about how event sourcing should be applied to just small,
appropriate domains within your organisation. He used the following advice from
the definition of a **domain model** in [Patterns of Enterprise Application
Architecture](https://www.amazon.co.uk/Enterprise-Application-Architecture-Addison-Wesley-Signature/dp/0321127420)
to suggest how you might identify somewhere where implementing event sourcing
might make sense:

> "Use where you have complicated and ever-changing business rules."

One other piece of guidance that Udi gave was, that event sourcing can be a good
fit if you have a bi-temporal domain - that is, _"something happened, and, it is
valid until…"_

## Event Source vs Event Driven Architecture

One common theme of the talks during the conference were people sharing the
lessons they'd learned and the mistakes that were made along the way. Amongst
this advice, there were some common themes.

![The Gartner Hype Cycle](/images/what-is-event-sourcing/tweet.png)

The most common theme that I noticed throughout the talks was that often people
confuse event sourcing with Event Driven Architectures. That is to say that the
events were viewed as a means of communication between services. Event sourcing
and Event Driven Architectures can be used together, but they are not the same
thing.

The big problem with merging the two concepts comes when you start publishing
your internal event sourced events as public messages to be consumed by other
services. Doing so couples the internal domain to the external messaging. You
may indeed want to store an `OrderPlaced` event and publish one to a message bus
for other services to consume, but different information might need to be in
these data structures. For example, the internal event stored event does not
need to include many details because they are already in the event history.
However, the `OrderPlaced` message might be the first public message to be
published about this order and should contain all the relevant details (i.e.
_order ID_, order line details, etc.)

## In Conclusion

Having a dedicated event sourcing conference is a great sign that event sourcing
is something you should know about. One thing that appeared on a few slides
during the conference was the Gartner Hype Cycle graph, with a suggestion that
many people are somewhere between the _Peak of Inflated Expectations_ and the
_Trough of Disillusionment_. Meanwhile, the community leaders are guiding us to
the _Plateau of Productivity_.

![The Gartner Hype Cycle](/images/what-is-event-sourcing/gartner.jpg)
Source: [gartner.com](gartner.com)

I hope this article has served as a useful and digestible introduction.

---

## References

[DDD Europe 2020](https://dddeurope.com/2020/)
[Slides from Udi Dahan's keynote](https://www.slideshare.net/udidahan/udi-dahan-event-sourcing-keynote-ddd-eu)
[The example code used in this article](https://github.com/tomphp/event-sourcing-example)
[Greg Young's CQRS/ES PDF](https://github.com/keyvanakbary/cqrs-documents) - a great introduction to both CQRS & event sourcing
[Event Sourcery](https://eventsourcery.com/) - a great event sourcing tutorial

---

Featured photo by
[geralt](https://pixabay.com/illustrations/time-time-management-stopwatch-3216244/) on
[Pixabay](https://pixabay.com/illustrations/time-time-management-stopwatch-3216244/).