---
title: TDD — A New Transformation
description: In this article, I question whether there is a transformation missing from the Transformation Priority Premise (TPP). In my last article…
featured_image: '/images/tdd---a-new-transformation/featured-image.jpeg'
---

In this article, I question whether there is a transformation missing from the
[Transformation Priority
Premise](http://blog.cleancoder.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html)
(TPP). In my [last
article](https://cloudnative.ly/tdd-lookahead-refactoring-53d677d4677f), I
implemented a simple function which checked whether the input number was odd or
even. The goal of the exercise was to strictly adhere to the TDD cycle while
also making sure I had all the tests I wanted at the end. This exercise lead to
an impasse problem, where I couldn’t write the next failing test without
changing the behaviour. To get beyond this impasse, I changed the behaviour in
the _refactor_ stage and claimed that it was OK to do so because it enabled the
next step in the unfinished process.

In this article, I want to do the same exercise, but this time a refactor must
truly be a refactor — i.e. not change the behaviour in any way. Also, to get
from _red_ to _green_, I will strictly stick to the Transformation Priority
Premise.

### The Transformation Priority Premise

The Transform Priority Premise, or TPP, is the idea that by applying
transformations with specific priorities during the TDD cycle, we can end up
with superior solutions.

A transformation is a change we make to the code to get a failing test to pass;
it’s the dual of a refactoring:

* **Refactoring** — Changing the structure of the code without changing the behaviour.
* **Transformation** — Changing the behaviour of the code without changing the structure.

The list of transformation as defined by Robert Martin are (order is from
highest to lowest priority):

> 1. **({}–>nil)** no code at all->code that employs nil
> 2. **(nil->constant)**
> 3. **(constant->constant+)** a simple constant to a more complex constant
> 4. **(constant->scalar)** replacing a constant with a variable or an argument
> 5. **(statement->statements)** adding more unconditional statements.
> 6. **(unconditional->if)** splitting the execution path
> 7. **(scalar->array)**
> 8. **(array->container)**
> 9. **(statement->recursion)**
> 10. **(if->while)**
> 11. **(expression->function)** replacing an expression with a function or algorithm
> 12. **(variable->assignment)** replacing the value of a variable.

The idea with the TPP is always to use the highest priority transformation to
get from _red_ to _green_.

### The Exercise

Let’s begin the exercise, the rules are:

1.  Strictly follow the TDD cycle
2.  Always use the TPP to get from _red_ to _green_
3.  End with two tests for the even case and two for the odd case (see [previous article](https://cloudnative.ly/tdd-lookahead-refactoring-53d677d4677f))

#### Step 1

We start with the following test:

```ruby
it 'returns false for 1' do
  expect(even?(1)).to be(false)
end
```

And we use the **(nil -> constant)** transformation to get the following
implementation (Ruby implicitly returns **nil**, so we don’t need to use **({}
-> nil)**):

```ruby
def even?(_number_)
  false
end
```

#### Step 2

We create the next failing test:

```ruby
it 'returns true for 2' do
  expect(even?(2)).to be(true)
end
```

To make this pass we have to apply **(unconditional -> if)**:

```ruby
def even?(_number_)
  return true if _number_ \== 2
  false
end
```

I could refactor the code here to `number == 2`, but I’m going to leave it for
now.

#### Step 3

We cannot write a failing test for the number **3** as it will already pass, so
let’s move on to **4**:

```ruby
it 'returns true for 4' do
  expect(even?(4)).to be(true)
end
```

Looking at the TPP, the only transformation I can apply to go green is
**(unconditional -> if)** again:

```ruby
def even?(_number_)
  return true if _number_ \== 2
  return true if _number_ \== 4
  false
end
```

At this point, I want to generalise the code as having two special case
conditions is making the code very specific (remember that the idea with TDD is
_as the tests get more specific, the implementation should become more
generic_). The problem, however, is that any attempt to generalise this code
will result in changes to the behaviour — which violates our refactoring rule.

Given we can’t generalise using either a refactoring or one of the defined
transformations, the only failing tests we can write are to test even
numbers — which will just drive more **if** statements. What can we do?

#### Adding a New Transformation

This exercise suggests to me that maybe the list of transformations is
incomplete, so I want to try adding a new one. I’m going to suggest that
transformation **5.5** is **(if expression -> generalised if expression)**.

> …
> 5\. **(statement->statements)** adding more unconditional statements.
> 5.5. **(if expression -> generalised if expression)** generalise the predicate expression in an if condition.
> 6\. **(unconditional->if)** splitting the execution path
> …

With this new transformation, let’s go back to the beginning of step 3. At this
point we have the following passing tests:

```ruby
it 'returns false for 1' do
  expect(even?(1)).to be(false)
end

it 'returns true for 2' do
  expect(even?(2)).to be(true)
end
```

The following implementation:

```ruby
def even?(_number_)
  return true if _number_ \== 2
  false
end
```

And we add the following failing test:

```ruby
it 'returns true for 4' do
  expect(even?(4)).to be(true)
end
```

To make this pass, we apply the new **(if expression -> generalised if
expression)** transformation (which importantly is a higher priority than
**(unconditional -> if)**):

```ruby
def even?(_number_)
  return true if _number_ \>= 2
  false
end
```

This time I am going to refactor:

```ruby
def even?(_number_)
  _number_ \>= 2
end
```

#### Step 4

Interestingly, this generalisation has created a situation where we can now
write a test for the next odd input:

```ruby
it 'returns false for 3' do
  expect(even?(3)).to be(false)
end
```

Finally, by applying the same **(if expression -> generalised if expression)**
transformation, we can complete the implementation:

```ruby
def even?(number)
  number % 2 == 0
end
```

### Closing Thoughts

Should **(if expression -> generalised if expression)** be added to the list of
transformations in the TPP? I’m currently unsure, but I certainly feel there’s
a case for consideration. I stated in my previous article that _often when
doing TDD we’re not just trying to implement the final solution, but we’re also
trying to create a test suite which gives us confidence_. This new
transformation allowed us to achieve that in the above example (while also very
strictly sticking to the TDD cycle and definitions of refactoring and
transforming).

This has been another fun little TDD thought experiment, I hope you have
enjoyed following along.

[^1]: Image by [FranckinJapan](https://pixabay.com/en/users/FranckinJapan-2774010/) via [Pixabay](https://pixabay.com/en/robot-transformer-gundam-tokyo-1464596/)
