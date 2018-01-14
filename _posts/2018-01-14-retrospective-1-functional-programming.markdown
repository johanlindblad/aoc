---
layout: post
title:  "Retrospective 1: Functional programming (and Elixir) is great"
date:   2018-01-14 10:00:00 +0100
categories: retrospective
---
My biggest (and most general) takeaway from this year's (by now, last year's) AoC is that functional programming is great.
My only real project written functionally before was a Clojure game we did as project work in a course. While I already before
then liked functional programming as a concept and tried to incorporate similar patterns even in imperative code, I never quite
got over the famous parenthesis-soup of the LISP paradigm. In a similar way I had dabbled in Haskell before but never got over the monad thing. 

Doing Elixir for these 25 days was a great way to avoid these obstacles for a while and instead get into the functional way
of thinking. I had to overcome the challenges inherent in writing functional code such as the inability to mutate state and
requirement of passing all parameters explicitly, without having to also overcome syntactical challenges or learn category
theory.

Since finishing day 25 I have started experimenting with [Elm](http://elm-lang.org) and am slowly making myself comfortable
with the syntax and strong typing. I feel that being comfortable with Elixir has made this process a lot easier.

## Favorite things
1. Pattern matching. I will really miss this when writing in languages which don't support it. It is a great way to separate the general case from boundary conditions and makes code so much cleaner. I am especially happy about [Day 9](/aoc{% post_url 2017-12-09-day9-stream-processing %}), where this was put to use.
2. Lack of accidental mutation of state. I have experienced 0 bugs due to accidentally overwriting state (or trying to but instead mutating a copy). This would be unconceivable in for example Ruby, which I used in previous years.
3. Multiple declarations. Taking care of special cases in their own function definition makes the general case so much simpler to both read and write. I miss this in languages without support (such as Elm) although pattern matching still helps somewhat.

## Least favorite things
1. Lack of constant-time arrays. There are a few days where a simple solution is simply not possible because there are no regular arrays. Obviously this problem follows logically from the definition of functional programming and is to be expected; often there are other, more functional ways of achieving the same thing. Furthermore there are tree-based alternatives which are `O(log n)` and often work well enough.

I really cannot think of any other things. This has been great.
