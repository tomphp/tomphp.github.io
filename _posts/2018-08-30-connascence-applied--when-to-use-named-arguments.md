---
title: 'Connascence Applied: When to Use Named Arguments'
description: I recently found myself doing some work in Python and started questioning when I should be using named function arguments. After pondering…
featured_image: '/images/connascence-applied--when-to-use-named-arguments/featured-image.png'
---


I recently found myself doing some work in Python and started questioning when
I should be using [named function
arguments](http://www.diveintopython.net/power_of_introspection/optional_arguments.html).
After pondering for a while on personal preference, I realised that there was a
more factual answer based on the ideas of
[Connascence](http://connascence.io/). In this article, I introduce these
concepts and demonstrate one way in which connascence can help you write better
code.

### Named Arguments

In Python, you can choose to call any function with or without named arguments.
For example — given a function defined like so:

```python
def send_message(email_address, subject, message)
```

You can choose to call the function with positional arguments like so:

```python
send_message("riend@example.com",
             "Hello Friend",
             "Dear Friend, I hope you are well")
```

Alternatively, you can choose to use named parameters:

```python
send_message(email_address="friend@example.com",
             subject="Hello Friend",
             message="Dear Friend, I hope you are well")
```

_An important fact about named arguments is that you don’t have to provide them
in the correct order._

When it comes to choosing which style you want to use, you can come up with
various arguments based on coding style and preference. However, by considering
connascence, a software quality metric &amp; a taxonomy for different types of
coupling, you can make a much more educated decision.

### Connascence

Connascence is a way to think about, identify and discuss different types of
coupling in software. I was re-introduced to connascence in a session with [Ian
Johnson](https://twitter.com/IJohnson_TNF) at
[SoCraTesUK](http://socratesuk.org/) this year. Since taking it back to
Armakuni, it has become a relatively common part of mine and
[Billie](https://medium.com/u/58c48d3546e)’s language at work (it’s
particularly useful when working with legacy code).

Connascence is made up of three parts:

#### Locality

Locality is the distance between coupled pieces of code. A private method
defined and called only within a single class is very local. A public method
which is called from entirely different modules/areas in the codebase has a
much lower locality.

#### Degree

The degree is the number of times a specific coupling occurs. If a method is
only ever called once then the degree is very low. If it’s called many times,
then it’s high.

#### Strength

Connascence identifies different types of coupling and puts them in order of
strength — the stronger the coupling the harder the code is to maintain. These
strengths are also grouped into two categories —  _static_ and _dynamic_.

Static types of connascence can be recognised by reading the code. These are
(from weakest to strongest):

* [Name](http://connascence.io/name.html)
* [Type](http://connascence.io/type.html)
* [Meaning](http://connascence.io/meaning.html)
* [Position](http://connascence.io/position.html)
* [Algorithm](http://connascence.io/algorithm.html)

The dynamic types of connascence are all stronger than the static types. They
are identified by analysing the runtime behaviour of the code and are harder to
recognise/understand. These are (from weakest to strongest):

* [Execution](http://connascence.io/execution.html)
* [Timing](http://connascence.io/timing.html)
* [Value](http://connascence.io/value.html)
* [Identity](http://connascence.io/identity.html)

#### How does Connascence help?

To write maintainable code, we want to reduce coupling as much as possible. By
designing our code so that the coupling is as low and weak as possible, we make
it much easier to change and move forward. Connascence gives us a metric to
identify when and how we can improve our code to achieve these goals.

The idea is that we want to lower the strength of any coupling we find — and
the higher the degree and lower the locality, the more important it is to
reduce the strength. If we can’t lower the strength, then we want to reduce the
degree and/or increase the locality.


_For the rest of this article, I’ll only be talking about the connascences of
name and position. To learn about all the other type see
[http://connascence.io/](http://connascence.io/)_.

To determine when to use named arguments, let’s first understand the _name_ and
_position_ types of Connascence:

#### Connascence of Name

Connascence of name is the lowest strength type of coupling in the list. In
fact, connascence of name is present in almost every single line of code — and
therefore is generally not a negative type of connascence.

Connascence of name is simply that one piece of code refers to another piece of
code by name; this could be a class name, variable name, method name or an
argument name. It’s coupled because if you change the name in one place, you
have to change it in the other place too.

```python
def say_hello():
  print(&quot;Hello&quot;)

def greet():
  
say_hello()
```

#### Connascence of Position

Position is the second highest static connascence. It exists when you have to
have additional knowledge about the order that things appear in. For example:

```python
name = row[0]
age = row[1]
favourite_colour = row[2]
```

If the code that creates `row` changes the order of the the elements, then
you’d have to consider updating every piece of code that referred to the
elements in `row` by index.

In the case of function arguments, connascence of position occurs anywhere that
you call a function with positional arguments:

```python
def send_email(email_address, subject, message):
  # implementation...

send_mail("friend@example.com", "Hello Friend", "I miss you")
```

If you change the order of the arguments in the function signature, then you
need to update every place which calls it.


> **Aside:**
>
> In a compiled, statically typed language, the compiler would help identify
> these changes — and this can be improved further by using specific types
> rather than primitives. However, in dynamically typed languages it’s more
> important to make it clear so that mistakes are recognised before going into
> production.
>
> Note: Using specific types instead of primitives is moving from connascence
> of meaning to connascence of type. Connascence of type is a lower strength
> than meaning, so this is a good thing.

It is possible to describe the order of the arguments in the name of the
function. e.g.

```python
users = fetch_users_by_age_and_favourite_colour(23, "red")
```

However, this runs the risk of the parameter order being changed but the name
not being updated. It’s also overly verbose.

### So when should we use named arguments?

Surely, given all this information about connascence the answer is to always
use named arguments — right?

Using named arguments definitely provides some benefits:

* Easier to understand when reading function calls
* No need to worry about changing the parameter order
* Adding a new argument is easier (especially if it has a default value)

However, it does add verbosity to the code which may be less desirable in some
situations. Therefore there are some situations when using positional arguments
might be the better option. These are:

#### 1. When you have a low degree and high locality

If a function/method is only ever called from within the file it is defined in
(assuming that file is not too large), then it might be more readable to just
positional arguments:

```python
def fizzbuzz(number):
  if _is_divisible_by(3, number) and _is_divisible_by(5, number):
    return "fizzbuzz"

  if _is_divisible_by(3, number):
    return "fizz"

  if _is_divisible_by(5, number):
    return "buzz"

  return number

def _is_divisible_by(divisor, number):
  return number % divisor == 0
```

#### 2. When you have a single argument function

If a function takes only a single argument and is unlikely to ever have to take
more, then its parameter may be so obvious that it is not necessary to use the
name.

```python
result = square_root(number)
```
#### 3. When the order is implicit in the domain context

Sometimes a domain context has an implicit order. In this case, I think it can
be OK to rely on the positional arguments.

```python
red = rgb_colour(255, 0, 0)
```

### Conclusion

Connascence is a simple idea that can help us write better code. In this case
we’ve learned that favouring named arguments in languages which allow them
creates more maintainable code.

I intend to follow this article up with another one which looks at how this
applies in a strictly typed language like Haskell — taking into account the
connascences of _type_ and _meaning_.
