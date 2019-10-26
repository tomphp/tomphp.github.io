---
title: TDD — Choosing the Right Intermediate Steps
description: In this article, I show the importance of considering the right steps to make when doing TDD. I show that a seemingly simple addition to…
featured_image: '/images/tdd---choosing-the-right-intermediate-steps/featured-image.png'
---

In this article, I show the importance of considering the right steps to make
when doing TDD. I show that a seemingly simple addition to the code might
introduce more complexity than expected — leaving code which is not covered by
tests. I also show how taking the wrong steps can get in the way of the
_red-green-refactor_ cycle of TDD.

### An Example

I’m going to use an example. The aim is to create a Twitter-like social network
application, where the user has a feed of messages. Messages appear in the
user’s feed under two circumstances:

1. If someone they follow posted the message
2. If someone posts a message that mentions them

A third rule we want to implement is that a particular message should only
appear in the feed once (i.e. if someone followed by a user, also mentions the
user, then the message should only appear in the user’s feed once).

### Getting Started

We’re going to do this with strict TDD. The first few iterations get us the
code below (I have implemented this as a single static method in an attempt to
simplify the code examples):

#### Tests

```java
public static final Set<String> NO_FOLLOWEES = new HashSet<>();

@Test
public void return_an_empty_list_if_there_are_no_message()
{
    ArrayList<Message> messages = new ArrayList<>();

    List<Message> feed = Messages.createFeed("Alice", NO_FOLLOWEES, messages);
    
    assertThat(feed).isEmpty();
}

@Test
public void returns_messages_posted_by_followed_users()
{
    Message message = new Message("Bob", "Hi from Bob");
    List<Message> messages = Arrays.asList(message);

    Set<String> followees = following("Bob");
    List<Message> feed = Messages.createFeed("Alice", followees, messages);

    
    assertThat(feed).contains(message);
}

private Set<String> following(String user)
{
    Set<String> followees = new HashSet<>();
    followees.add(user);

    return followees;
}

```

#### Implementation

```java
public static List<Message> createFeed(
        String user,
        Set<String> followees,
        List<Message> allMessages
) {
    List<Message> result = new ArrayList<>();

    for (Message message : allMessages)
    {
        if (message.isByOneOf(followees)))
        {
            result.add(message);
        }
    }

    return result;
}
```

### Writing the Next Test

At this point, we have tested and implemented the first requirement — the user
can see messages from people they follow.

The next requirement is to show the user messages which mention them. Here’s
the test:

```java
@Test
public void returns_messages_which_mention_the_user()
{
    Message message = new Message("Bob", "Hi @Alice");
    List<Message> messages = Arrays.asList(message);

    List<Message> feed = Messages.createFeed("Alice", NO_FOLLOWEES, messages);

    assertThat(feed).contains(message);
}
```

At this point, I want to explore two different ways to make the test pass.

#### 1. Adding an OR condition

```java
public static List<Message> createFeed(
        String user,
        Set<String> followees,
        List<Message> allMessages
) {
    List<Message> result = new ArrayList<>();

    for (Message message : allMessages)
    {
        if (message.isByOneOf(followees) || message.mentions(user))
        {
            result.add(message);
        }
    }

    return result;
}
```

This seems like a simple step — the test is now green, and the requirement to
show messages which mention the user is implemented.

> Skipping the _red_ step of the _red-green-refactor_ cycle is not good — if
> you don’t see the test fail then you cannot be confident that it is correct.

However, I’d like to draw your attention to the fact that we’ve also
implemented the final requirement. If there is a message mentioning a user, and
the user is following the author, then the user will only see one version of
the message.

It might seem good that we’ve implemented all the requirements, but we have no
explicit test covering the last one. We can write a new test to cover that
requirement, but it will pass straight away. Skipping the _red_ step of the
_red-green-refactor_ cycle is not good — if you don’t see the test fail then
you cannot be confident that it is correct.

Let’s now look at what we could have done instead.

#### 2. Add a Second if Statement

If instead of writing the code above, we had written the following code, then
what would have happened?

```java
public static List<Message> createFeed(
        String user,
        Set<String> followees,
        List<Message> allMessages
) {
    List<Message> result = new ArrayList<>();

    for (Message message : allMessages) {
        if (message.isByOneOf(followees)) {
            result.add(message);
        }


        if (message.mentions(user)) {
            result.add(message);
        }

    }

    return result;
}
```

In this case, we have also made the test pass, and implemented the requirement.
However, we have not implemented the third requirement. In this case, if a user
has been mentioned and they follow the author, then they will see the message
appear twice. This now lets us write the next test:

```java
@Test
public void not_add_a_message_to_the_feed_more_than_once() {
    Message message = new Message("Bob", "Hi @Alice");
    List<Message> messages = Arrays.asList(message);

    List<Message> feed = Messages.createFeed(
            "Alice", following("Bob"), messages);

    assertThat(feed).containsOnlyOnce(message);
}
```

Now we can watch this test fail, and then update the code to the working
implementation we saw previously. However, this time we have all the tests:

```java
public static List<Message> createFeed(
        String user,
        Set<String> followees,
        List<Message> allMessages
) {
    List<Message> result = new ArrayList<>();

    for (Message message : allMessages)
    {
        if (message.isByOneOf(followees) || message.mentions(user))
        {
            result.add(message);
        }
    }

    return result;
}
```

### What if the Requirement Was Different?

An interesting thing happens if we change the requirement. If instead, we
wanted the message to appear in the feed twice, then the problem would be
reversed. This shows that no approach is ultimately more correct than the
other; instead, we see that choosing the right intermediate steps to take when
doing TDD is context specific.

### Conclusion

When doing TDD, you have two goals:

1. To drive the development of the implementation
2. To ensure that your test coverage is comprehensive

To choose the right intermediate steps to write, you have to keep both of these
goals in mind. Using a climbing wall as a metaphor — our aim is not just to get
to the top, but to also find a way to explore every climbing hold on the way
up.

One thing that can help us ensure that we take the right steps is the
[Transformation Priority
Premise](https://8thlight.com/blog/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html);
this guides on the best way to get from _red_ to _green_. The premise, however,
does not cover the example we have just seen. The choices that the transform
priority premise don’t give the answers to are where the context-specific
decisions have to be made.

Finally, I’d like to point out that it’s important to recognise when a step is
an intermediate one. The goal of each intermediate step is to drive us to take
the next step. We want to make sure that the steps we take explore the full
scope of the solution.

I hope you have learned something from reading this article, I most certainly
learned a few things writing it.
