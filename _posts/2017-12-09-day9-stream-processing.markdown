---
layout: post
title:  "Day 9: Stream Processing"
date:   2017-12-09 12:00:00 +0100
categories:
tags: easy
---
# Part 1

Today we are asked to parse a string which contains:
* Groups, beginning and ending with `{` and `}` respectively
* Garbage, beginning and ending with `<` and `>` and containing random garbage
inbetween. Furthermore characters can be escaped by `!`, causing them not to
count.

The first question regards the number of groups; more specifically the sum of
the depths of the appearance of all groups (i.e. `{ { { } } }` gives `1+2+3=6`).

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day9.Part1 do
  def score(string), do: elem(parse(string), 0)

  def parse(string, level \\ 1, score \\ 0, garbage \\ 0)
  def parse("{" <> rest, level, score, garbage), do: parse(rest, level + 1, score + level, garbage)
  def parse("}" <> rest, level, score, garbage), do: parse(rest, level - 1, score, garbage)
  def parse("," <> rest, level, score, garbage), do: parse(rest, level, score, garbage)
  def parse("<" <> rest, level, score, garbage), do: garbage(rest, level, score, garbage)
  def parse("\n" <> rest, level, score, garbage), do: parse(rest, level, score, garbage)
  def parse("", _, score, garbage), do: {score, garbage}

  def garbage(">" <> rest, level, score, garbage), do: parse(rest, level, score, garbage)
  def garbage("!" <> rest, level, score, garbage), do: escaped(rest, level, score, garbage)
  def garbage(<<_>> <> rest, level, score, garbage), do: garbage(rest, level, score, garbage + 1)

  def escaped(<<_>> <> rest, level, score, garbage), do: garbage(rest, level, score, garbage)
end
{% endhighlight %}

I think this solution turned out really well, and really highlights the
strengths of pattern matching. It is basically a 
[recursive descent parser](https://en.wikipedia.org/wiki/Recursive_descent_parser)
where all the logic is encoded in the pattern matching itself. The last time I
wrote a parser such as this I did it in Python and that was not as easy to read.

# Part 2
In part two we instead ask how many characters of garbage there are, not
counting any escaped ones. The solution for part 1 solves this problem as well
so the only code is a function to extract the right element from the tuple:

{% highlight elixir %}
defmodule Aoc.Day9.Part2 do
  import Aoc.Day9.Part1
  def garbage_count(string), do: elem(parse(string), 1)
end
{% endhighlight %}

# Final thoughts
This was a fun little problem which didn't take too long to solve. This was the
first one where I really thought Elixir was a great match as it allows the
solution to be really elegant.

This was also the first day where I thought of using default values for
arguments in Elixir, making future code slightly more clean.
