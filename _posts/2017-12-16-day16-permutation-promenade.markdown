---
layout: post
title:  "Day 16: Permutation promenade"
date:   2017-12-16 12:00:00 +0100
categories:
tags: easy revisit
---
# Part 1
This problem concerns simple "dance moves" that modify a list of the characters
`a` to `p` in some way.

They will either:
1. Rotate the list (spin)
2. Exchange two given indices
3. Exchange two given values

My solution just simulates these instructions exactly as given:

{% highlight elixir %}
defmodule Aoc.Day16.Part1 do
  import Aoc.Day10.Part1, only: [rotate: 2]
  def start(), do: [:a, :b, :c, :d, :e, :f, :g, :h, :i, :j, :k, :l, :m, :n, :o, :p]

  def preprocess(input), do: input |> String.trim |> String.split(",")

  def run(instructions, programs \\ start()) do
    Enum.reduce(instructions, programs, &perform/2)
  end
  def perform("s" <> rest, programs), do: spin(programs, String.to_integer(rest))
  def perform("x" <> rest, programs) do
    [first, second] = rest |> String.split("/") |> Enum.map(&String.to_integer/1)
    exchange(programs, first, second)
  end
  def perform("p" <> rest, programs) do
    [first, second] = rest |> String.split("/") |> Enum.map(&String.to_atom/1)
    partner(programs, first, second)
  end

  def spin(programs, steps), do: rotate(programs, -steps)

  def exchange(programs, i1, i2) do
    value1 = Enum.at(programs, i1)
    value2 = Enum.at(programs, i2)
    programs |> List.replace_at(i1, value2) |> List.replace_at(i2, value1)
  end

  def partner(programs, value1, value2) do
    i1 = Enum.find_index(programs, &(&1 == value1))
    i2 = Enum.find_index(programs, &(&1 == value2))

    programs |> exchange(i1, i2)
  end
end
{% endhighlight %}

# Part 2
In part two we are to repeat these operations a billion times. This is not even
close to be feasible if it is simply simulated. Instead, we find that the
instructions given lead to a cycle. Because there is no other state than the
contents of the list, we know that the same list will always lead to the same
result if given the same instructions.

For this reason we can simply find out when when the cycle starts and how long 
it is, and then reduce the number of iterations from 1 billion to a much smaller
number that is at the same position in the cycle.

{% highlight elixir %}
defmodule Aoc.Day16.Part2 do
  import Aoc.Day16.Part1, only: [start: 0]
  alias Aoc.Day16.Part1

  def solve(instructions) do
    {repeat_start, cycle_length} = repeats_after(instructions)
    reduced = rem(1_000_000_000 - repeat_start, cycle_length)

    run(instructions, reduced) |> Enum.join("")
  end

  def repeats_after(instructions) do
    Stream.iterate(0, &(&1 + 1))
    |> Enum.reduce_while({start(), %{}}, fn(index, {programs, previous}) ->
      case Map.get(previous, programs) do
        nil -> {:cont, {Part1.run(instructions, programs), Map.put(previous, programs, index)}}
        prev_index -> {:halt, {prev_index, index - prev_index}}
      end
    end)
  end

  def run(instructions, times) do
    Stream.iterate(start(), &(Part1.run(instructions, &1)))
    |> Enum.at(times) 
  end
end
{% endhighlight %}

# Final thoughts
This was a quite fun problem and I really like that the given instructions hint
at there being more to a solution than just simulating it. If we were asked to
just do a million iterations I might not have thought of it.
