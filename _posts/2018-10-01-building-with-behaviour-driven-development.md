---
title: Building With Behaviour-Driven Development
description: Building software with Behaviour-Driven Development is like building a house. Actually, it’s not — but it’s a metaphor I want to explore…
featured_image: '/images/building-with-behaviour-driven-development/discovery.png'
---

### Building With Behaviour-Driven Development

Building software with Behaviour-Driven Development is like building a house.
Actually, it’s not — but it’s a metaphor I want to explore. The motivation for
this post comes from the fact that every person I talk to seems to have a
different idea of what BDD is. This article is simply me sharing my personal
view of what I think it is.

* * *

### Some Things I’ve Heard

I've heard BDD described in many different ways, some of these include:

*   Using Cucumber is doing BDD
*   BDD is a communication tool
*   BDD is like TDD but talking about behaviours instead of implementation
*   Gherkin feature files are _BDD Tests_
*   You are either doing BDD or TDD; you can’t do both

I have some pretty strong opinions on each of these points, but I don’t want to
address them directly. Instead, I want to paint a picture of my view of how a
project is run using BDD.

### Discovery, Formulation, Automation

The BDD community is now starting to adopt the idea that
[discovery](http://bddbooks.com/), formulation and automation are the three
tools of BDD.

#### Discovery

![Dicovery](/images/building-with-behaviour-driven-development/discovery.png)

I’m going to liken the discovery phase to the initial planning stages of
building a house. During this time we decide what we want to build and work out
if the features are viable. We think about how many rooms we want, how many
floors, do we have enough room to build a garage, and how much budget do we
have. In the software context, this is where we think about what it is that we
really want to build.

#### Formulation

![Formulation](/images/building-with-behaviour-driven-development/formulation.png)

Formulation is where we take what we have discovered and create some
specifications. For my metaphor, I’m going to compare this with the creation of
architectural plans and working out which materials are needed. In the context
of BDD, this would be the use of discovered examples to create scenarios.

#### Automation

![Automation](/images/building-with-behaviour-driven-development/automation.png)

The word automation feels a bit wrong to me here. It suggests that the aim of
this phase is only to create automated tests, when in fact, I see it as driving
the development of the system using automated tests. For the metaphor, I’m
relating this to physically building the house.

### Test-Driven Development — Building Walls

![Building with TDD](/images/building-with-behaviour-driven-development/tdd-walls.png)

When doing TDD, you’re thinking at the code level, considering each line you
write. Practising TDD is like building a wall; you are concentrating on laying
each single brick.

> **BDD for unit testing**
>
> I think it’s worth mentioning that there are BDD testing frameworks for unit
> testing. These frameworks make use of words like **expect** and **should**
> instead of **assert**. These frameworks aim to get the developers to specify
> the behaviour of the code rather than the implementation. However, for me,
> this is just TDD with extra emphasis on the language (which is a good thing).
> Therefore, when I’m talking about BDD, I’m usually assuming we’re talking at
> the feature level, rather than the code level.

### Putting it all Together

#### Building houses without plans

You can build houses without plans. It’s much better to have plans though, this
way you can work out the details earlier in the process rather than when it’s
too late.

_We can create working software without BDD. However, BDD is a way to help us
work out what the business requirements are, and it enables us to build the
right thing collaboratively._

#### Architects can’t create the plans alone

The architects need to collaborate with the property owner and other
construction experts to create the plans. They have to work with the owner to
realise the vision of the house they want. They have to work with other experts
to understand the materials that can be used and what can be done on the plot
of land where the house will be built.

_Our software specifications cannot be created by one person alone. It’s true
that the architect might draw the plans alone, or a developer might write the
scenarios alone, but this has to be done using information gathered
collaboratively in the discovery phase. They also can be iterated on._

#### Plans are readable by everyone

Architectural plans can be used and understood by everyone. The architects use
them to design and plan, the owner of the house uses them to check the house
they want will be built, and the builders use the plans to actually build the
house.

_The plans are like our software specifications (feature files). All the
parties involved should be able to read them and find them useful._

#### Walls without house

We can build walls without them being part of houses.

_We don’t need to be doing BDD to do TDD._

Sometimes these walls are useful — _we create working software._

Other times we build walls just to learn more about the process of building
walls — _code katas_.

#### Houses without bricks

We can build houses without brick walls (e.g. log cabins). However, brick walls
are often the best choice.

_We can build software without TDD. However, using TDD is often the best
choice._

#### Building the walls is an important part of building houses

When we’re building the walls of a house, we focus on the process of building
walls, but we are still in the process of building the house.

_While we are writing code we are focussing on the code and TDD; however, while
we are doing this we are still building a product using BDD._

### Final Thoughts

This is a somewhat abstract post which compares chalk with cheese. However, I
hope it shows that BDD is a large scale process which includes everyone. The
other aspects of software development (such as TDD) don’t go away — they are
all part of the process of implementing BDD.

BDD covers the whole process — from initial conception to final implementation.
In my opinion, BDD is not a tool or framework, but rather a mindset which you
employ at all levels in your project.

* * *

### Metaphor Issues

* Architectural plans are not used to test the construction of the house (they are not executable).
* In general, construction requires more big design up front and software is more iterative. (i.e. one set of plans are drawn up at the earlier stages of a construction project, but in a software project new specifications are written up throughout the development process).
