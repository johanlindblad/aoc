---
layout: post
title:  "Day 13: Packet Scanners"
date:   2017-12-13 12:00:00 +0100
categories:
tags: easy revisit
---
# Part 1
Today's problem concerns layers in a firewall, which can be passed as long as
the detector is in the right place. Essentially each layer counts from 0 up to a
specific "depth" and back again; it can be passed as long as it is not at 0.

We are asked for the sum of "products" of the places where we would be caught
(which is calculated as its depth times its number).

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day13.Part1 do
  def solve(input) do
    input |> preprocess |> travel |> Enum.map(&tuple_product/1) |> Enum.sum 
  end

  def preprocess(input) do
    input |> String.trim |> String.split("\n") |> Enum.map(&preprocess_row/1)
  end
  def preprocess_row(row) do
    [l, d] = row |> String.split(": ") |> Enum.map(&String.to_integer/1)
    {l, d}
  end

  def travel(layers, initial_delay \\ 0) do
    Enum.filter(layers, &(caught_at(&1, initial_delay)))
  end

  # A layer with n depth takes n steps to the bottom and then n - 2 steps
  # until it gets back up
  def caught_at(layer_def, delay \\ 0)
  def caught_at({layer, depth}, delay), do: rem(layer + delay, (depth*2) - 2) == 0

  defp tuple_product({a,b}), do: a * b
end
{% endhighlight %}

# Part 2
In part 2 we are wondering how long we must wait at the beginning to be able to
pass safely. I solved it like this:

{% highlight elixir %}
defmodule Aoc.Day13.Part2 do
  alias Aoc.Day13.Part1

  def solve(input), do: input |> Part1.preprocess |> required_delay

  def required_delay(layers, delay \\ 0) do
    case Enum.any?(layers, &(Part1.caught_at(&1, delay))) do
      true -> required_delay(layers, delay + 1)
      false -> delay
    end
  end
end
{% endhighlight %}

It is pretty straight-forward. It seems to run faster than some other people I
saw. Breaking an iteration as soon as we are caught the first time likely really
helps (`Enum.any?/2` is of use here).

# Final thoughts
This was a fun problem which I might revisit later. The "flavor" of part 2 is
has hints of the (Chinese Remainder Theorem)[https://en.wikipedia.org/wiki/Chinese_remainder_theorem]
and I might try to see if it can be solved in a similar way. I still have code
from my cryptography course that I can use.
