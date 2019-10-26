---
title: 'Designing Microservice Architectures for People'
description: Right now, microservice architectures are everywhere — everyone is doing them. However, it seems that many organisations which have…
featured_image: '/images/designing-microservice-architectures-for-people/featured-image.jpeg'
---

Right now, microservice architectures are everywhere — everyone is doing them.
However, it seems that many organisations which have invested in designing and
building new microservice architectures are struggling to scale their
development teams. A fundamental reason for this is that technology has been
the primary focus of the design when it really should have been people — let me
try and explain...

As a developer (or architect) tasked with the job of building a microservice
architecture for the first time, the first thing you do is go searching for
information. Given the complexity of creating a microservice architecture, it’s
only natural that the biggest question that comes to mind is “How do we do
this?”. When searching, it’s not long before you find yourself neck deep in
information about _message queues_, _REST_, _circuit breakers_, _eventual
consistency_ and many other technical topics and patterns which are vital to
the creation of resilient microservice architectures.

The problem occurs when implementing technical design patterns becomes the
driving forces in the architecture design. If a system is built this way, then
you can end up with a technically focused architecture. This makes it hard to
scale the teams because the system was never designed with that in mind.

The reason I’m writing this article is because I believe a lot of hassle can be
avoided upfront by concentrating on _why?_ rather than _how?_

### So why do you want to build microservices?

In [episode 141](http://blog.cognitect.com/cognicast/141) of the
[Cognicast](http://blog.cognitect.com/cognicast/), Michael Nygard eloquently
states that he views microservices as a _technological solution to an
organisational problem_. He then goes on to explain that the reason we use them
is because as the team that builds the monolith grows, productivity goes down.
So, the reason why we use microservices is to separate the functionality of the
product into separate services — then smaller teams can simultaneously work on
separate services without disrupting each other.

When trying to design microservices, I’ve seen a fairly common pattern where
people try to separate microservices along technical boundaries (e.g. the
front-end the back-end, the mailing system, etc.). The problem with separating
the services and teams along these technical boundaries is that all the teams
have to be involved when implementing any new feature.

Also, any product of any scale has multiple types of user that interact with
it. Each of these groups of users will want separate sets of features
implemented with competing priorities. The result of all this is multiple teams
try to deal with competing requests coming from multiple places, while they are
all working on a single system. This doesn’t work well.

### Building microservices for people

To make microservice teams scale, the teams and services need to be autonomous.
They must be able to work and deploy at their own individual rate — not limited
by conflicts with other teams or services. To achieve this autonomy, I
recommend focusing on three primary ideas:

* _The Single Responsibility Principle_
* _Bounded Contexts_
* _Conway’s Law_

By using any one of these to drive your design, you are likely to end up adhering to the others. By continuously considering all three you stand a very good chance of success!

#### The Single Responsibility Principle

The Single Responsibility Principle (SRP) comes from the SOLID principles for object-oriented programming. Even though the SRP is related to OOP, people reference it in many other contexts. Often they use it to mean 
_do one thing and do it well_ (see the 
[Unix Philosophy](https://en.wikipedia.org/wiki/Unix_philosophy#Do_One_Thing_and_Do_It_Well)). However, this doesn’t quite capture the true nature of the principle — in Robert C. Martin’s blog post titled 
[The Single Responsibility Principle](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html) he states the following:

> This principle is about people.
>
> When you write a software module, you want to make sure that when changes are
> requested, those changes can only originate from a single person, or rather,
> a single tightly coupled group of people representing a single narrowly
> defined business function. You want to isolate your modules from the
> complexities of the organization as a whole, and design your systems such
> that each module is responsible (responds to) the needs of just that one
> business function.
>
> - Robert C. Martin

If we now apply this meaning to microservices, we are saying that each
microservice should only change based on requirements coming from a single
business function.

![The Single Responsibility Principle](/images/designing-microservice-architectures-for-people/srp.jpeg)

#### Bounded Contexts

Within the domain of a company, different people (or business functions) have
specialised views of the domain. For example, an accounting department might
talk about _invoices_, whereas the shipping department doesn’t care about them.
Also, they most likely both view and interact with _customers_ in different
ways.

Bounded Contexts are a strategic modelling tool from Domain-Driven Design. The
idea is that rather than creating a single model which tries to aggregate the
views of all business functions, you create independent models for different
views — each of these models exist inside a Bounded Context.

In general, the boundaries of the Bounded Contexts tend to align with the
boundaries of the subdomains. If you then align your microservice boundaries
with the Bounded Contexts boundaries, then you should naturally adhere to the
SRP.

![Bounded Contexts](/images/designing-microservice-architectures-for-people/bounded-contexts.jpeg)

#### Conway’s Law

> organizations which design systems … are constrained to produce designs which
> are copies of the communication structures of these organizations.
>
> - Melvin Conway

Conway Law’s suggests that however hard you try to create a system where the
architecture doesn’t match the communication structures in the organisation,
eventually the architecture will evolve until it does. The reason for this is,
as different business functions are competing for features to be developed, the
developers will start to segment them into different areas of the system so
that they can be worked on independently.

Designing systems with Conway’s Law in mind from the start can significantly
help understand where the Bounded Contexts are. It also enables the development
of autonomous microservices and teams from the beginning.

![Conway's Law](/images/designing-microservice-architectures-for-people/conways-law.jpeg)

### In conclusion

Microservices primarily bring organisational benefits, but they introduce
significant technical challenges. If a microservice architecture does not
deliver those organisational benefits, then they only introduce unnecessary
technical challenges.

When designing microservice architectures, you must keep in mind that the key
reason for doing so is people. By starting with separate teams developing
independent microservices which serve separate business functions, you enable
autonomy from the beginning.

In addition to this, I would recommend starting with just one microservice per
Bounded Context. Others can be extracted out only when there is a good reason
to do so (such as long-running processes, services are getting too big, or to
match new business requirements). This way, you defer all the additional
complications that microservices bring until you know you need to face them.

Happy microservicing!
