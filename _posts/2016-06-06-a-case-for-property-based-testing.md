---
title: A Case for Property Based Testing
category: testing
layout: post
tags:
    - testing
    - property based testing
    - clojure
    - tdd
    - socratesuk
---

At SoCraTes UK this year, a group of us wanted to try and understand how to use
Property Based Testing effectively. We had a great discussion during the day,
then attempted to TDD the Diamond Kata with it during a mob programming session
in the evening. Ultimately we never completed the exercise, but I think all
involved learnt a lot in the process. Even so, we never reached a conclusion to
our original question - *Should property based testing be used to do TDD, and
if so, then how?*

Since the event, I've been thinking about it quite a lot. At this point, I
don't think Property Tests should replace Unit Tests in TDD. However, I do
believe that I've found the type of situation where using Property Tests is
useful - to drive further development by discovering the *edges* of your
implementation.

## The Prime Numbers Kata

I'm going to use the Prime Numbers Kata in Clojure to demonstrate my point.

At this first point, I had used Unit Test based TDD to arrive at, what I think,
is a complete solution.

The tests looked like this:

```clojure
(deftest prime-factors-test
  (testing "1 returns an empty vector"
    (is (= (prime-factors 1) [])))
  
  (testing "2 returns just 2"
    (is (= (prime-factors 2) [2])))
  
  (testing "3 returns just 3"
    (is (= (prime-factors 3) [3])))
  
  (testing "4 returns 2 and 2"
    (is (= (prime-factors 4) [2 2])))
  
  (testing "6 returns 2 and 3"
    (is (= (prime-factors 6) [2 3])))
  
  (testing "8 returns three 2s"
    (is (= (prime-factors 8) [2 2 2])))

  (testing "9 returns 3 and 3"
    (is (= (prime-factors 9) [3 3]))))
```

And the implementation looked like this:

```clojure
(defn factor? [divisor number] (zero? (mod number divisor)))

(defn- prime-factors-loop
  ([divisor number]
   (cond
    (= number 1) []
    (= divisor number) [number]
    (factor? divisor number) (concat [divisor] (prime-factors-loop divisor (/ number divisor)))
    :default (prime-factors-loop (inc divisor) number)))) 

(defn prime-factors [number] (prime-factors-loop 2 number))
```

At this point, the tests all passed. And I couldn't think of another example to
write which would cause the tests to fail. That to me was an indicator that
maybe I had completed the algorithm - but I wanted to be sure.

## A New Test

To put it to the test, I decided to use a much larger number and ensure it gave
the correct result. Now rather than work out the expected result for my large
input number, I worked backwards by selecting a list of primes and multiply
them together to get the input number to test against.

The test I created looked like this:

```clojure
  (testing "the prime factors of the product of a list of prime numbers is those numbers"
    (is (= (prime-factors (* 2 2 3 5 7 13 13 17)) [2 2 3 5 7 13 13 17])))
```

Sure enough, the test passed. I felt happy and confident that I had completed
my implementation.

However, there was something interesting to notice here -the description of
this last test is in some way different to all the others. All the previous
tests each explained a single, focused example of a behaviour of the function.
But this new test explains a rule which holds for the whole implementation.
Could this rule be a good candidate for property test?

## The Property Test

Using the `clojure.test.check` library, I created the following property test
(`prime-numbers` is a vector containing the first 1000 prime numbers).

```clojure
(defspec the-prime-factors-of-the-product-of-a-list-of-prime-numbers-is-those-numbers
  100 ; Number of iterations to run.
  (prop/for-all [primes (gen/vector (gen/elements prime-numbers))]
    (= (prime-factors (apply * (map bigint primes))) (sort primes))))
```

This test makes `100` attempts at running `prime-factors`, each time with a
number generated from a random list of prime numbers. The result is then
compared to the sorted list to confirm the behaviour.

Feeling confident in the algorithm, I ran my tests. However, this was the
result:

```
lein test property-testing-example.core-test
{:result #error {
 :cause nil
 :via
 [{:type java.lang.StackOverflowError
   :message nil
   :at [clojure.lang.Numbers$LongOps isZero "Numbers.java" 443]}]
 :trace
 [;...
 ]

Ran 2 tests containing 9 assertions.
0 failures, 1 errors.
Tests failed.
```

My recursive solution had blown the stack! That's certainly something I don't
want to happen. With this new bit of information, I then updated my function
to be tail call recursive (by using Clojure's `loop`/`recur` construct):

```clojure
(defn factor? [divisor number] (zero? (mod number divisor)))

(defn prime-factors [input-number]
  (loop [factors []
         divisor 2
         number input-number]
    (cond
     (= number 1) factors
     (= divisor number) (conj factors number)
     (factor? divisor number) (recur (conj factors divisor) divisor (/ number divisor))
     :default (recur factors (inc divisor) number))))
```

Finally, I have a successful implementation:

```
lein test property-testing-example.core-test
{:result true, :num-tests 100, :seed 1465248402934, :test-var "the-prime-factors-of-the-product-of-a-list-of-prime-numbers-is-those-numbers"}

Ran 2 tests containing 9 assertions.
0 failures, 0 errors.
```

## Conclusion

Now it's perfectly possible that I could've written an explicit example cause
the stack to blow. However, to have created such an example, I would have had
to have been focussing on the implementation of the language. What's more, at
some point my code might be compiled on a platform with a different stack
size - invalidating such an example. **By using a Property Test, I managed to
locate an implementation edge case error by focusing only on the domain of the
problem I was solving,** for this reason I can see this as being a valuable
tool!

### An Admission

For this example, I did intentionally set out to blow the stack. However, when
I wrote the property test, I initially failed to generate an error because my
range of input prime numbers was not high enough (only the first 10). After
increasing the list, I got the error I was expecting. This raises the question
that if you don't know what error you are looking for, how do you ensure that
the range that your generator is generating is sufficient? Something to ponder
on...

This topic is a new adventure for myself and many others, if you have
experience in or thoughts about this subject, we'd be very grateful if you
would share them.
