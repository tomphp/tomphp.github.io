---
title: Should You Test Private Methods?
category: TDD
layout: oldpost
tags:
    - tdd
    - php
---

While trying to learn TDD I found myself asking this question fairly often.
I've searched for the answer online many times and always got mixed responses.
I have also devised many clever ways to try to test them, and even ways to
mock $this. However, now I've got to a point where I'm confident I have a very
valid answer to this question and I'm going to try to explain it here...

## So, "Should you test private methods?"

The short answer is **No!**.

The slightly longer answer is *"If you are even asking this question you do not
understand TDD yet."* That might sound a little harsh but I will explain why
this is in a moment. Before I do that I just want to say that I can think of
one valid reason where you might want to test a private method, I will
explain that at the end of this post.

## The Long Answer

### Where to private methods come from?

The first thing I want to talk about here is where private methods come from in
the TDD cycle.

The first thing you do is write your test, you then try to satisfy that test.
Usually to satisfy a test it only takes a few lines of code added, or
modifications to the method, or part of the code that you are testing.
Generally you will do these additions/modifications inline.

Once the test passes it's time to refactor. This is generally the point at
which you'll take some slightly complex looking bits of code, or nested
condition/loop statements, and move them to one or more private methods. At
this point your now have neater and more descriptive looking code. On top of
that **your private method is already tested by the tests covering the code
calling it**.

### Surely it's easier to test something once rather than multiple times?

The next question is, what if multiple methods call a private method? Surely
then it's easier to test the private method once, rather than once for each
method calling it?

My answer to that is again "**No**". The reason is: when using TDD you
should be building your code incrementally, you should be satisfying a test by
calling the private method rather than calling it and working out how to test
it. The call to the private method should solve the failing test, not the other
way around.

Another possibility here might be that the private method is doing something
suitably complex. Something that writing the failing test (or tests) that would
be fixed by calling it from would be quite a long and complex process. In this
situation your private method is probably doing a specific task, different from
the focus of the class it exists in. This breaks the *Single Responsibility
Principal*. In this case it should refactored out into its own class which can
be independently tested.

## What about protected methods? And should I test abstract classes and traits directly?

OK, so now things get a bit different. Here I think it depends on the purpose
of the methods in question, as well as how they came into existence...

### For abstract classes and traits which for part of a library's API

If you are writing a library and you create an abstract class or a trait for
people who use that library to extend or use then my answer is *"Yes,
absolutely."*.

A good way to do this is to create a test class in your test suite which
uses/implements the trait/abstract class, allowing you to test the public
methods. You can also  add any public methods needed to expose the protected
methods (private methods still need not be tested independently as they are not
part of the API). Again, the methods should be created because of the tests,
not tests to try to test the methods.

### For parent classes or traits which have appeared out of the development process

Often a parent class or trait won't be something you decide to come up with
first, but rather as the tests drive you to develop a class (or maybe a 2nd
class that has similar functionality to an existing class). You decide that
maybe moving some methods to a parent class or trait would be a good idea.

When you do this, the tests which cover the class you have extracted methods
from, will still cover your new parent class/trait so do you need to test them
independently?

I think to answer this question is that you need to decide what the purpose of your
new parent class/trait is. If it is to provide a shared bit of functionality for
one or more classes in your system, but will unlikely be used by other people
then I think it is fine to leave it as - being covered by the tests for the
classes using it. However, if it is to become part of your API and you expected
other people might like to use/extend it then yes, I think it should be tested
independently. This way should you ever remove the classes which you have
written that use or extend your trait or parent class, then it will still be
tested.

*As a little bonus point: I think that if you have tested a parent class
independently, then you can remove the tests from the children which test the
functionality of the parent class, then simply test that the child classes
extend from the tested parent class. This is personal preference but I like to
do this.*

*So my main point here is: if something forms part of an API (including public
methods, protected methods and traits) then they should be tested
independently.*

## When should I test a private method?

At the beginning I said that there was one situation when I though maybe
testing a private method might be a good idea. This is when you have a legacy,
un-tested system, which you want to try to add some tests to ensure you don't
break anything when trying to refactor/update it. In this case, I think certain
situations might present themselves where this is a good, temporary idea.
However, in the long run I would hope that eventually the need for that test
will be refactored away.

## Conclusion

So my main points in this article are:
* If you are doing TDD right, you should never find yourself asking if you
should test a private method.
* Private methods, and in fact all code, should be created because of the tests
- not the other way around.
* Make sure that anything that makes up part of your API is tested
independently! Everything you write should be tested anyway!

Finally, this is an informed opinion, taken from what I have learnt from
research and experience. It's working perfectly for me but if you have a
different view on this I'd love to hear about it!

Also, if you think I have not been clear about anything or would like some code
examples, then let me know!
