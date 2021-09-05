
# Beyond Test-Driven Development

Photo by ThisisEngineering RAEng on Unsplash

I’m a firm believer that Test-Driven Development (TDD) should be the default way that software teams create code. I believe that its methodical approach leads developers to design and discover better ways to architect their software. However, I also believe that the best tool that we have today is not guaranteed to be the best tool for tomorrow. In this article, I present some of the things that I’ve explored which could perhaps replace TDD (and maybe even tests in general) as the best way to build and test software. This article does not seek to provide any answers, but rather, present some exciting things to play with.
> “Technologies, methodologies, and practices are constantly emerging and evolving. There is always a better way of doing things. Although we described a set of practices in this chapter, that neither means they are the only practices out there, nor that they are suitable in every single context. Things that are considered good today may not be considered as good tomorrow. Maybe in a few years time, Software Craftsmanship will be strongly encouraging the adoption of a completely different set of practices”
> [Software Craftsmanship](https://www.amazon.co.uk/Software-Craftsman-Professionalism-Pragmatism-Robert/dp/0134052501) by Sandro Mancuso

## About TDD

### What is it?

[Test-Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) is a precise process. It is the quick iteration of the *red-green-refactor* cycle. In this cycle, we think about what we need to achieve, write a single failing test (red), write **just enough** code to make that test pass (green), and then refactor to improve the design. Each iteration of this cycle takes only a few minutes (or less).

There are many benefits to TDD:

* It gives the developer a rigorous process which makes it easy to know what to do next.

* It breaks the problem down into much smaller pieces; this reduces cognitive load and allows the practitioner to maintain the state of flow.

* It encourages decoupled and cohesive, modular design in your code.

* It results in near 100% code coverage without even making that the focus (it should be 100%, but that’s dependant on the developers not accidentally missing a step).

### What is its current state?

The good news is that tests are much more common now than they used to be. Many projects have at least a few tests, and many developers are familiar with testing frameworks. Also, the words *Test-Driven Development* (and acronym *TDD*) feel like they are mainstream now, you can find them on many CVs and job descriptions. However, my experience tells me something a bit different about the state of TDD — having done countless interviews of people with TDD on their CV, I can count on one hand the number of people who I’ve seen do TDD by-the-book in an interview. What I see instead is people practising their own interpretation of the words “test-driven development”; this can range from processes very similar to TDD (perhaps writing a suite of tests first) to merely having some form of tests present at the end of the process.

My experience suggests that, while the term Test-Driven Development is well known, the practice still has a long way to go until it becomes mainstream. Whether TDD will ever become entirely ubiquitous for the industry or not, I do not know. However, I still think we should strive for this as a goal. TDD is, without a doubt in my mind, the most reliable way that we currently know of for writing software with the languages, tooling, processes and teams that are most common in the industry. **I am, however, also a firm believer that this will not always be true!**

### When is TDD Appropriate?

I think it’s also worth pointing out that TDD isn’t a tool appropriate for every single programming task; there are times when it is not the best tool for the job. Experienced TDD practitioners have explored TDD to a level where they know it’s limitations. They will know when they need to either change the process or switch processes altogether — however, this will generally be for a small part of the codebase rather than the whole project.

For general programming, TDD is usually applicable in my experience. When someone tells me that a piece of code is not able to be written using TDD, then nine times out of ten, I’ll disagree — often it requires a small change in mindset or approach. That one time I will agree though, it will be completely valid, and TDD is not the right tool.

I think it’s also worth pointing out that there are niche alternatives to TDD. These might exist in specific programming paradigms, domains or communities. These alternatives are very valid in their context; they are also some of the sources that this article draws from. However, in business software development, I stand by my statement that **TDD is currently the best tool for the job**.

Now we know where we are with TDD, I want to explore some different ideas…

## Evolutions of TDD

The most obvious processes that could succeed TDD are the direct descendants of it. Many of these are TDD with some restriction applied (e.g. time limitations within the cycle) or some adjustments/additions to the process (e.g. revert changes when the tests fail).

One of the most talked-about evolutions is [Kent Beck](undefined)’s [**test && commit || revert](https://medium.com/@kentbeck_7670/test-commit-revert-870bbd756864)** (or TCR). With this process, every time you run the tests, the changes are committed if the tests pass, or **reverted if the tests fail**. A typical first reaction to trying this is that it’s infuriating, but as you proceed, you see that there are lessons to be learned.

Personally, I’m not sold on this being a better process than TDD for the experienced practitioner, but I do see it as a fantastic training tool. *The reason I believe that it is so valuable as a learning tool is that it forces the person using it to discover their own ways to take smaller steps — *this self-discovery I find much more valuable and rewarding than just following instructions from a book or teacher.

Rachel M. Carmena has created some great diagrams of some of the different flows people have developed. You can find them in the article [Test-driven programming workflows](https://rachelcarmena.github.io/2018/11/13/test-driven-programming-workflows.html).

While all these altered workflows are interesting, I still see them as striving towards the same goal. I thoroughly recommend experimenting with these different workflows and seeing what you learn (you definitely will learn something). Still, I see them as the evolution of TDD rather than moving beyond TDD.

## REPLs

Many languages now come with a REPL (a “*read, evaluate, print loop”* or merely an interactive terminal). Python, Ruby and Scala, for example, all have tools which allow you to run commands and evaluate expressions interactively. The developers in communities from some languages, such as Clojure and F#, commonly use interacting with the REPL as their primary development process; that is, they iterate the development of functions by interactive tweaking and trying until they reach the desired implementation. You can call this [*REPL-driven developmen](https://www.google.com/search?q=repl+driven+development)t*.

An evolution of REPL usage is where IDEs have implemented features where you can evaluate lines and expression inline in the code editor. When it is as convenient as hitting something like Alt+Enter to get the result of evaluating the expression on the current line, you are getting some of the fastest feedback possible.

Another variation on this, which is often found in the academic, science and data-science domains, is the use of [Jupyter Notebooks](https://jupyter.org/) to create interactive and executable documentation. This documentation can even include things like generated graphs, tables and charts. Jupyter Notebooks combines the REPL like feedback in a [Literate Programming](https://wiki.haskell.org/Literate_programming) context. Interactive Notebooks can provide a fantastic record of the process behind the development of the code.

The big problem I have with the use of the REPL as the primary iteration process is that you still have to write the tests. TDD gives a great way to know which tests to write, but with the REPL, you have to decide which tests to create after the fact. The caveat here is that this apparent limitation might be due to inexperience on my part.

## Types

Let’s briefly talk about types. In languages where we can specify the types of values, we write fewer tests. For example, in Ruby we might write a test it "returns an integer", but in Java, we’d define public int f(). There is no need to write such as test in Java because we trust the type system (although tests around null are still very much needed in many languages).

One thing some functional programmers like to do in strongly typed languages is to start creating a model by defining the types. Once the types are specified, then the implementations can be written, along with tests, to complete the features. You can find a great example of this in [Scott Wlaschin](undefined)’s fantastic [Domain Modelling Made Functional](https://www.amazon.co.uk/Domain-Modeling-Made-Functional-Domain-Driven/dp/1680502549), which shows a practical way to apply this approach.

Some languages have even more powerful type systems that allow for *type-level programming. *As we’re now about to see, some languages have far more advanced type systems than our mainstream languages. These languages can reduce the amount of testing required to a much greater extent.

## Proofs

Since learning Test-Driven Development (I won’t say TDD in this section for reasons which are about to become apparent), I’ve been convinced that you must always have tests — that is until I read [Type-Driven Development with Idris](https://www.manning.com/books/type-driven-development-with-idris).

[Idris](https://www.idris-lang.org/) is a *dependently-typed language; t*his means that you can be much more expressive and restrictive in the type signatures of functions. This expressiveness is taken to a level at which the types of parameters and return values of functions can change and depend based on each other. This may sound complicated (it kind of is) but let’s look at a simple example. The following is a type signature for a function that joins two vectors together:

    join : (xs : Vect n elem) -> (ys : Vect m elem) -> Vect (n + m) elem

This type signature states that the function join does the following:

* It takes one vector (Vect) as the first parameter (named xs), of any length n, and containing elements of any element type elem.

* It takes another vector as the second parameter (named ys), of any length m, and containing elements of the same element type elem as in xs.

* It then returns a resulting vector containing elements of the same typeelem, **and it has the length of exactlyn + m**.

This example has taken one property of the algorithm to join two vectors together (i.e. “*the length of the resulting vector must be equal to the sum of the lengths of the two inputs*”) and specified that in the type. Now here’s the fun part, you don’t need to test this property of the algorithm because Idris’s compiler** will not compile this code until you prove to it that this property will always hold for your implementation**.

Obviously, in the example above, you still need tests as there are other properties of this algorithm that are not encoded in the type. These include:

* *The resulting vector begins with the first vector*

* *The resulting vector ends with the second vector*

It is possible with dependently typed languages to encode a lot more in the type. Some time ago, [Johan Martinsson](undefined) and I paired on a FizzBuzz implementation as an Idris kata; eventually, we came up with a solution, and the types looked like this:

    IsFizz : (k : Nat) -> Type
    IsFizz k = (modNatNZ k 3 SIsNotZ = 0)

    IsBuzz : Nat -> Type
    IsBuzz k = (modNatNZ k 5 SIsNotZ = 0)

    data Fizzbuzz : (k: Nat) -> Type where
     Fizz : (k: Nat) -> IsFizz k -> Not (IsBuzz k) -> Fizzbuzz k
     Buzz : (k: Nat) -> Not (IsFizz k) -> IsBuzz k -> Fizzbuzz k
     FizzBuzz : (k: Nat) -> IsFizz k -> IsBuzz k -> Fizzbuzz k
     Normal : (k: Nat) -> Not (IsFizz k) -> Not (IsBuzz k) -> Fizzbuzz k

This then enabled the following implementation:

    fizzbuzz : (k: Nat) -> Fizzbuzz k
    fizzbuzz k =
      let isFizz = decEq (modNatNZ k 3 SIsNotZ) 0
          isBuzz = decEq (modNatNZ k 5 SIsNotZ) 0
      in case (isFizz, isBuzz) of
           (Yes isfizz, No notbuzz) => Fizz k isfizz notbuzz
           (No notfizz, Yes isbuzz) => Buzz k notfizz isbuzz
           (Yes isfizz, Yes isbuzz) => FizzBuzz k isfizz isbuzz
           (No notfizz, No notbuzz) => Normal k notfizz notbuzz

Without going into detail of how this code works, the interesting point is that it is impossible to implement a function that does anything different than the implementation above with the type signature fizzbuzz : (k : Nat) -> Fizzbuzz k. Therefore, **there is no valid test that you can write for this function that would fail if the code compiled!**

Here I’ve shown that it’s possible (but not always practical) to write programs with Idris where tests are not needed, but Test-Driven Development is not just about the tests — it’s the methodology and process. Idris’s Type-Driven Development has its own process; it’s methodical just like Test-Driven Development, and it makes heavy use of the advanced IDE integration. Not only can Idris’s IDE integration/REPL evaluate statements, but it can also generate code and suggestions from the types. With Type-Driven Development, you specify a type and then work with the compiler to *define* and *refine* the implementation.

Dependent types and Idris’s great tooling seem near to perfection (although there are some more challenging gymnastics you have to do to convince the compiler that things are true), so why is everyone not doing it? The answer is that it’s still pretty academic, at the time I was playing with it (version 2 has since been released), it wasn’t mature enough or performant enough to write real-world applications with. It also requires some pretty radical changes in thinking from traditional software development. While there is some support for dependant and type-level programming in other languages, it’s still pretty niche and unknown — maybe this will change in the future.

## Property-Based Testing

In the last section, I talked about properties quite a bit. We’ve seen that dependently typed languages allow us to encode properties into types, but can we test properties in languages which don’t support dependant types? This is exactly what property-based testing does.

With TDD (I’ll revert to using TDD in place of **Test-**Driven Development from now on), we tend to use *example-based tests*. That is, we set up the environment, then execute the test with a specific set of inputs, and finally assert that we receive a specific result. Example-based tests are ideal for TDD because they are perfectly suited to small iterations. However, a single example rarely tells us much about the system under test on its own — instead you need a suite of examples to build up a model of *what you think* the system does (one of the skills of TDD is choosing the right examples).

With a properties, we don’t state that specific inputs give a specific result, rather we state that properties hold across sets of inputs and results.

* “*when given 6 return ‘fizz’*” is an **example**

* “*when given an input divisible by 3 return a result containing ‘fizz’*” describes a **property**

The problem with properties is that is is too expensive, impractical or even impossible to create tests which check every possible input. Instead, property-based testing frameworks use small random samples (say 100 or 1,000 sets of inputs).

Here’s an example of how you might test the above property might using Python’s [Hypothesis](https://hypothesis.readthedocs.io/en/latest/) framework.

    @given(x=integers())
    def test_input_divisible_by_3_returns_fizz(x):
        assert f(x * 3) == 'fizz'

Hypothesis will run this a specific number of times (e.g.100) with random integers provided for x. This means that when you implement f you will have to provide a solution which works for any integer value of x rather than just the specific one — otherwise, the test *could* fail.

Inspecting examples allow us to estimate the behaviour of an algorithm, whereas, with properties, we can specify the complete algorithm. A set of examples for Fizz Buzz might be:

* 1 == ‘1’

* 2 == ‘2’

* 4 == ‘4’

* 3 == ‘fizz’

* 6 == ‘fizz’

* 5 == ‘buzz’

* 6 == ‘buzz’

* 15 == ‘fizzbuzz’

* 30 == ‘fizzbuzz’

From these examples, we can deduce a good guess of the behaviour of the algorithm, but having these as tests would not stop the implementation from having an overriding case saying if x > 100: return 'flop'.

A set of properties for fizzbuzz might look like this:

* if input is divisible by 3, return a result containing ‘fizz’

* if input is divisible by 5, return a result containing ‘buzz’

* if input is not divisible by 3, return a result which does not contain ‘fizz’

* if input is not divisible by 5, return a result which does not contain ‘buzz’

* if input is not divisible by 3 or 5, return the input

Given this set of properties, there is no other implementation that holds true other than a correct one.

Property-based tests look much better than example-based tests, so why not always use them? When comparing property-based tests with example-based tests, property-based tests seem superior, but the problem with them is that the workflow is not as straightforward as when using example-based tests and TDD. For some tasks, doing TDD with property-based tests works perfectly (try the fizzbuzz example above), but for other tasks it’s not so obvious. You will often find that you end up either taking too big steps or using weak properties (rather than the ones which most characterise the algorithm) when trying to do TDD with property-based tests.

I find a combination of property-based and example-based tests works well. Sometimes it’s obvious when using a property makes sense, other times I’ll use example-based TDD and then refactor some of the example-based tests into property-based tests when I feel they are more appropriate.

## Algebra-Driven Design

I recently discovered the book [Algebra-Driven Designed](https://algebradriven.design/) by Sandy McGuire. What I found compelling about this approach is that it brings a methodology to using a collection of approaches (some of which I’ve mentioned already) together.

In the section about types, we talked about how some developers like to define the types as the first step when modelling in code. Sandy McGuire extends this a bit by creating the types **and then determining the laws which must be true in the model **(defining an algebra). In the book, Sandy builds a composable graphics library where a function called cw rotates an image tile 90 degrees clockwise. Its type signature takes a tile and returns a new tile:

cw :: Tile -> Tile

An example law for this is that “*if you rotate the tile 4 times, then you get the original tile”*. He represents this like so:

    ∀ (t :: Tile).
       cw (cw (cw (cw t))) = t

This is very much like defining a property which we talked about earlier.

The next step with Algebra-Driven Design is to implement a minimum viable version of the algebra (known as an *initial encoding*). This encoding implements all the functions in a way that obeys all the laws but isn’t necessarily useful.

Once this reference implementation is built, a tool called [QuickSpec](https://hackage.haskell.org/package/quickspec) is employed to **discover all the properties of the model!** These properties can be converted to property-based tests, and you are then free to create the implementation you desire.

The process of Algebra-Driven Development feels very different from TDD. You can define your algebra and laws with pen and paper while discovering the domain. The act of implementing is then creating the appropriate code within the laws of that algebra.

In the minimal experiments I’ve tried with Algebra-Driven Design, I’ve noticed that I tend to hit a well designed and composable API first time (as opposed to multiple refactorings before I get there). I’ve yet to be convinced that this approach can be used in all aspects required to build large systems, but it is definitely a completely different approach to that of writing code with TDD. I’m pretty sure Algebra-Driven Development excels with certain types of problem (the examples in the book are very good).

## Conclusion

This post has been quite a journey down the rabbit hole. It doesn’t contain any answers as to what will succeed TDD, but that was never my intent. In fact, I don’t believe that anything will succeed TDD; instead, we’ll discover new and exciting methodologies that can be used as complementary tools to support each other. Each of the things I’ve mentioned has their own strengths:

* TDD with examples excels in simplicity, methodology and test coverage.

* REPLs excel in the feedback cycle and discovery.

* Property-based testing excels in clarity, extended test coverage and edge-case detection.

* Type-level programming excels in correctness.

I don’t know how many of the things I’ve talked about will ever become mainstream programming; they are certainly not new (many of the ideas have been around more than a decade). I think that the reason they are becoming a bit more popular now is due to the rise in popularity of functional programming (where most of these ideas have come from). In the talk [Polysemy: Chasing Performance in Free Monads](https://www.youtube.com/watch?v=-dHFOjcK6pA), Sandy McGuire, says that the functional programming community is generally about 30 years ahead of the mainstream community — I have no idea if this is true, but perhaps some of these ideas will become mainstream one day.

In summary, learn TDD, practice TDD and teach TDD — I think we should all be trying to make TDD more common in our work. But also don’t be dogmatic — explore crazy ideas and alternatives and see what you learn or come up with. Finally, consider reading [Domain Modelling Made Functional](https://www.amazon.co.uk/Domain-Modeling-Made-Functional-Domain-Driven/dp/1680502549), [Type-Driven Development with Idris](https://www.manning.com/books/type-driven-development-with-idris) or [Algebra-Driven Design](https://leanpub.com/algebra-driven-design) this holiday.
