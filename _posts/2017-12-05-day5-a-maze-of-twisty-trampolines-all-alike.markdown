---
layout: post
title:  "Day 5: A Maze of Twisty Trampolines, All Alike"
date:   2017-12-05 12:00:00 +0100
categories:
tags: medium
---
# Part 1
Today we are asked to sort out interrupts from the CPU. Scaling back the layers
in the problem description we are left with a list of jumps to take through the
very same list. We will start at the beginning of the list, follow the jumps and
increase each element we end up at. When we finally try to jump to an index
outside the list, we are interested in how many steps we have taken.

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day5.Part1 do
  def steps_to_exit(input_string) do
    input_string
    |> String.trim
    |> String.split("\n")
    |> Enum.map(&String.to_integer/1)
    |> simulate()
  end

  @doc """
  Simulates the given instructions and returns the number of steps
  ## Example
  iex> Part1.simulate([0, 3, 0, 1, -3])
  5
  iex> Part1.simulate([-2, 1])
  1

  """
  def simulate(instructions), do: simulate([], instructions, 0)
  def simulate([], [instruction | _next], steps) when instruction < 0, do: steps + 1
  def simulate(previous, [instruction | next], steps) do
    {previous, next} = step(previous, [instruction + 1 | next], instruction)

    case next do
      [_head | _tail] -> simulate(previous, next, steps + 1)
      [] -> steps + 1
    end
  end

  @doc """
  Shuffles instructions from next to previous times times
  ## Example
  iex> Part1.step([2,1,0], [3,4,5], 2)
  {[4, 3, 2, 1, 0], [5]}
  iex> Part1.step([2,1,0], [3,4,5], -2)
  {[0], [1,2,3,4,5]}
  """
  def step(previous, next, 0), do: {previous, next}
  def step(previous, [instruction | next], times) when times > 0 do
    step([instruction | previous], next, times - 1)
  end
  def step([instruction | previous], next, times) do
    step(previous, [instruction | next], times + 1)
  end
end
{% endhighlight %}

In order to be able to get acquainted more with pattern matching I made a
recursive solution which always kept the current instruction at the head of its
list. All previous instructions will instead be in a separate list. When we
jump, we shuffle instructions back and forth between these two lists. The test
cases for `step/3` illustrate this. Because prepending to linked lists is `O(1)`
while appending is `O(n)` the first list is in the opposite order to how it is
to be interpreted.

With the `step/3` function written all we need to do is simulate the jumps by
shuffling back and forth according to the instructions and seeing at which step
we try to either escape the list to the left (first instance of `simulate/3`) or
to the right (case statement in second `simulate/3`).

# Part 2
For part two a very slight adjustment is made to how instructions are
incremented; instead of increasing an index unconditionally we instead decrement
it by 1 if it is greater than 2.

The code ends up being very similar:
{% highlight elixir %}
defmodule Aoc.Day5.Part2 do
  alias Aoc.Day5.Part1

  def steps_to_exit(input_string) do
    input_string
    |> String.trim
    |> String.split("\n")
    |> Enum.map(&String.to_integer/1)
    |> simulate()
  end

  @doc """
  Simulates the given instructions and returns the number of steps

  ## Example

  iex> Part2.simulate([0, 3, 0, 1, -3])
  10
    
  """
  def simulate(instructions), do: simulate([], instructions, 0)
  def simulate([], [instruction | _next], steps) when instruction < 0, do: steps + 1
  def simulate(previous, [instruction | next], steps) do
    {previous, next} = step(previous, [new_offset(instruction) | next], instruction)

    case next do
      [_head | _tail] -> simulate(previous, next, steps + 1)
      [] -> steps + 1
    end
  end

  defdelegate step(previous, next, times), to: Part1

  defp new_offset(old_offset) when old_offset >= 3, do: old_offset - 1
  defp new_offset(old_offset), do: old_offset + 1
end

{% endhighlight %}

# Final thoughts
This was a fun problem to solve. The way I shuffled lists around is probably not
the easiest way to do it but I had fun writing it. This is one of the few
problems I have left where my solution is so slow that I exclude it from my
regular test suite runs.

I might rewrite this solution to avoid doing the shuffling back and forth, as it
is unfortunately just as inefficient as it was interesting to write.
