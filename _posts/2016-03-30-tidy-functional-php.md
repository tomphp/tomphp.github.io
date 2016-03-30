---
title: Tidy, Functional PHP
category: php
layout: post
tags:
    - php
    - functional programming
    - transform
    - pentothal
---

When you start looking into functional programming and functional languages, it's
only a matter of time before you come across the `map`, `reduce` and `filter`
functions. Using these functions you can reason about performing actions on
collections as a whole, rather than concerning yourself about the internal
detail of looping over elements. As you get used to using these functions, you
start to see a real elgance in the way they describe the processing of data.

## Map, Reduce and Filter in PHP

PHP provides versions of these functions for when you are working with arrays.
These are `array_map`, `array_reduce` and `array_filter`. I use these a lot in
my PHP code, but there is something which bothers me about them. Before I go
into this, let's look at when we would choose to use each one.

### array_map

`array_map` is used to take one array, and create a new array with a one-to-one
mapping of each element processed via a transformation function. Take this
example:

```php
$names = [];
foreach ($customers as $customer) {
    $names[] = $customer->getName();
}
```

The `array_map` version looks like this:

```php
$names = array_map(
    function ($customer) {
        return $customer->getName();
    },
    $customer
);
```

### array_reduce

`array_reduce` uses a calculation function to reduce the values of all the 
elements into a single value. For example:

```php
$total = 0;
foreach ($items as $item) {
    $total += $item->getPrice();
}
```

The `array_reduce` version looks like this:

```php
$total = array_reduce(
    $items,
    function ($total, $item) {
        return $total + $item->getPrice();
    },
    0
);
```

### array_filter

`array_filter` is uses a predicate function to filter out unwanted items
from an array. For example:

```php
$redCars = [];
foreach ($cars as $car)  {
    if ($car->getColour() === RED) {
        $redCars[] = $car;
    }
}
```

Using `array_filter` would look like this:

```php
$redCars = array_filter(
    $cars,
    function ($car) {
        return $car->getColour() === RED;
    }
);
```

## The Problem

One of the nice things about using these functions is that they usually result
in neat, concise code. Take the above examples in the following languages:

#### Ruby

```ruby
names = customers.map(&:name)
total = items.reduce(0) { |total, item| total + item.price }
red_cars = items.select { |car| car.colour == :red }
```

Note `select` is the name Ruby uses for `filter`.

#### Clojure

```clojure
(def names (map :name customers))
(def total (reduce (fn [total, item] (+ total (:price item)))))
(def red-cars (filter #(= (:colour %) :red)))
```

#### Scala

```scala
val names = customers.map(customer => customer.name)
val total = items.foldLeft(0)((total, item) => total + item.price)
val redCars = cars.filter(car => car.color == Red)
```

(`reduce` is often also called `fold`).

As you can see, in most other languages uses of `map`, `reduce` and `filter`
are neat, single liners. However, in PHP the overly verbose anonymous function
syntax makes them pretty ugly and complicated looking. There was a
proposed RFC for a
[short closure syntax](https://wiki.php.net/rfc/short_closures) for PHP, which
would have enabled use to write nice had code like this:

```php
$names = array_map($customer ~> $customer.getName(), $customers);
$total = array_reduce($items, ($total, $item) ~> $total + $item.getPrice);
$redCars = array_filter($cars, $car ~> $car->getColour() === RED);
```

Sadly the RFC was rejected.

## A Solution - Transform and Pentothal

After spending quite some getting time frustrated with the PHP syntax, I
decided to do something about it. I started creating a library of functions
which each returned a closure to be used as a predicate for `array_filter` or a
transformation for `array_map` (I use `array_reduce` much less often).

After starting work, I doubted that I was be the first person to do this,
so I put it out to Twitter. [Giuseppe Mazzapica](https://twitter.com/gmazzap)
responded saying that he had a library which provides predicate part of what I
was doing - he then promptly upload his excellent
[Pentothal](https://github.com/Giuseppe-Mazzapica/Pentothal) library. From that
point on, I decided to just focus on the transformation part - I called this
library [Transform](https://github.com/tomphp/php-transform).

### Transform

Transform provides functions which create closures to perform many common
actions which you might commonly want to use with `array_map`. Taking our
example above you can use the following code:

```php
use TomPHP\Transform as T;

$names = array_map(T\callMethod('getName'), $customers);
```

*More details about [Transform](https://github.com/tomphp/php-transform) on GitHub.*

### Pentothal

In the same way, Pentothal provides many functions for generating predicate
closures to use with `array_filter`. Again, taking our earlier example:

```php
use Pentothal as P;

$redCars = array_filter($cars, P\methodReturn('getColour', RED));
```

*More details about [Pentothal](https://github.com/Giuseppe-Mazzapica/Pentothal) on GitHub.*

Both of these libraries provide a lot of options for most common requirements.
By using them the need to write a custom closure for `array_map` or
`array_filter` is pretty rare.

In conclusion: it looks like PHP 7.1 might finally get a decent, short closure
syntax added ([PHP RFC: Arrow
Functions](https://wiki.php.net/rfc/arrow_functions)). But in the meantime, I
hope you find these libraries useful.
