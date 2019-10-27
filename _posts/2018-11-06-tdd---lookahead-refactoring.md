---
title: TDD — Lookahead Refactoring
description: In this article, I want to consider a situation where considering the next test we want to write, influences the way we refactor.
featured_image: '/images/tdd---lookahead-refactoring/featured-image.jpeg'
---

In this article, I want to consider a situation where considering the next test
we want to write, influences the way we refactor.

> **Aside**
>
> This article is the third one I’ve written recently looking at interesting
> little nuances in the TDD process (see [TDD — Choosing the Right Intermediate
> Steps](https://cloudnative.ly/tdd-choosing-the-right-intermediate-steps-576e6526d1bd)
> and [Which Order to Write Your
> Tests](https://cloudnative.ly/which-order-to-write-your-tests-7ea2937761a1)).
> It was not my intention to create this mini-series on the topic, but it has
> been an exciting outcome of working on creating training material to teach
> TDD from the ground up. During this work, I have been learning these
> delightful new things — not in complex code, but in the simplest of problems
> possible.
>
> I consider myself a somewhat seasoned TDD practitioner, but through
> discovering new things in the absolute basics (where it would seem that there
> is nothing left to learn) highlights the importance of continually reviewing
> the foundations of our skills.

For this article, I want to consider implementing a simple function to check if
a number is odd or even. The function will simply return _true_ if it’s given
an even number, and _false_ if it’s given an odd number.

As with my previous TDD articles, we are working very rigidly with the TDD
cycle — we **must** write a failing test, we **must** implement it in the
simplest way possible, we may then refactor.

### The Example

For this example, let’s ignore the fact that Ruby’s built-in `Integer#even?`
method exists.

#### Step 1

We start with the following test:

```ruby
it 'returns false for 1' do
  expect(even?(1)).to be(false)
end

We make it pass by simply returning _false_:

def even?(_number_)
  false
end
```

#### Step 2

We add the next failing test:

```ruby
it 'returns true for 2' do
  expect(even?(2)).to be(true)
end
```

We make it pass by adding a guard clause:

```ruby
def even?(_number_)
  return true if _number_ \== 2

  false
end
```

And then we refactor:

```ruby
def even?(_number_)
  _number_ \== 2
end
```

#### Step 3

Now things get interesting — I’ve been working up through the numbers, but
writing a test for 3 (which would be the next logical progression) will not
fail. I want to stick very rigidly to the TDD cycle, so wehave two options:

1.  Choose an even number for my next test
2.  Refactor to allow us to write the next failing test

The first option might seem like the obvious one, but this article is about
investigating the second one. Before writing the next failing test, let’s
refactor again:

```ruby
def even?(_number_)
  _number_ != 1
end
```

At this point, the tests still pass. Now we can write the next failing test:

```ruby
it 'returns false for 3' do
  expect(even?(3)).to be(false)
end
```

Once we have seen it fail, we can write the code to make it green:

```ruby
def even?(_number_)
  _number_ != 1 && _number_ != 3
end
```

Now it's time to refactor. We could implement our final solution now but I
don't like that idea. We currently have a lack of symmetry, we have two tests
for the odd result but only one for the even — that bothers me. I want to add
another test for the even case but I can’t because we can’t write one which
will fail (I want to see it fail because it proves I’ve written the right
test). So what can we do?

#### Step 4

Let’s refactor again:

```ruby
def even?(_number_)
  _number_ \== 2
end
```

Again, the current tests pass, so now we can write the next failing test:

```ruby
it 'returns true for 4' do
  expect(even?(4)).to be(true)
end
```

Finally, we can complete the implementation:

```ruby
def even?(_number_)
  _number_ % 2 == 0
end
```

### That’s not refactoring!

Technically some of the refactorings I did above are not refactorings; they
change the outward behaviour of the function. However, in the context of tested
behaviour, they are refactorings.

What I want to point out here is that when you use TDD to create new code,
there is a different state of mind — we know we are in the process of creating
something new, and that we are not finished yet.

**Note:** I have since followed this up with [a second
article](https://cloudnative.ly/tdd-a-new-transformation-bb61bdee8422) which
looks at another way to achieve this while sticking strongly to the definition
of refactoring.

### Why?

What I have tried to highlight here is that often when doing TDD we’re not just
trying to implement the final solution, but we’re also trying to create a test
suite which gives us confidence.

Sometimes we can do TDD absolutely by the book and arrive at the solution we
want, but feel we are missing some tests. In this case, the temptation is often
to just add the additional tests, but when we do this we never see them fail.
For me, I’m almost more concerned with seeing a test fail than seeing it pass
because I want to be confident that I haven’t made an error in the test code.

> when we are practising TDD we must keep thinking ahead about the solution we
> are designing towards, even though were are focusing on each individual step
> along the way.

What this example shows, is that during our refactoring stage, it can help to
consider what we are trying to do next. This ties in with the same message I
was trying to highlight in my previous articles — when we are practising TDD,
we must keep thinking ahead about the solution we are designing towards, even
though we are focusing on each individual step along the way.

I hope you found this little thought exercise interesting.

[^]: Featured image: Image by [pixel2013](https://pixabay.com/en/users/pixel2013-2364555/) via [Pixabay](https://pixabay.com).
