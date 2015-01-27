---
title: Using Modelling By Example to Develop Understanding of New Programming Languages
category: modelling-by-example
layout: post
tags:
    - modelling-by-example
---

Right now, Konstantin Kudryashov's Modelling By Example is the most exciting
topic in my life. I've been thinking about it, playing with it and talking
about it with anyone who will listen. The funny thing is, the idea is kind of
obvious. The first thing I thought when I read about it was "Wow! That's
great." Shortly followed by "This is so obviously, surely everyone should
already be doing this!". That for me, shows what a great idea it is - if
something is so clearly obvious when it's pointed out to you, then it's got to
be a great idea.

As I started exploring it, my initial thoughts where all about how it could
help the way I currently work, and how it might improve my current approach to
building systems. This yielded many possible benefits. However, the more I
thought about it, I started to think of another interesting idea.

In my day job I'm primarily a PHP programmer, but I'm always looking at
learning other new and exciting languages to expand my knowledge of
development.  Most recently I've been trying to develop my functional mindset,
which took me briefly to Haskell, then to Scala and now to Clojure (which I'm
finding extremely exciting).

When learning a new programming language, after getting a grasp of the syntax,
I set myself 2 tasks. The first is to learn the recommended and idiomatic
approach to writing in the language. The second is how to transfer my existing
knowledge of software design to the language. The first task is very good, it
lets me learn the mindset of the language and what sort of task it's suited
for. The second however, is more tricky. It often results in writing code in
the style of the language I'm currently used to using, but in this new
language. This inevitably results in several bad attempts, missing the benefits
of this new language, before I start to actually get an idea of how I should be
using it. This process also involves many web search for things like "domain
driven design in clojure" or "repository pattern in clojure" - there are
generally people with ideas and suggestions, but there's a definite variation in
quality (especially with newer languages).

This is where I started to realise that Modelling By Example could provide
something very interesting here. Konstantin is pushing the idea that *your
scenarios are your model*. This means that your model can be represented in a
format that you're used to using, regardless of what programming language you
are working in. Once you have some scenarios, it's then your job to rewrite
each line of your test as a code - using the *ubiquitous language* to keep the
meaning as similar as possible. If you write your test code this way, keeping
it in the recommended, idiomatic style of the programming language, then this
starts to show you how to develop your model/functions. Sure, there will be many
more hurdles to overcome, but it should set you up with a very good start.

So, that's what's going on in my brain at the moment. It's worth pointing out
this is mostly theory, I've only experimented with this a little bit so far,
but it seems to make sense. The main caveat I can think of is: depending on
which programming language you are using, it may cause some feedback into the
way you structure the wording in the scenarios - however, I do think that this
might increase your understanding of the new programming language, rather than
cause a hindrance.

This is just an idea, but I'd love to hear your thoughts or experience.
