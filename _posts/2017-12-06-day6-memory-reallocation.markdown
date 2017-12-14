---
layout: post
title:  "Day 6: Memory Reallocation"
date:   2017-12-06 12:00:00 +0100
categories:
tags: easy
---
# Part 1
Today we are asked to reallocate memory banks. Essentially this boils down to
setting the largest element to 0 and distributing it as instances of 1 to
successive indices (for example the list `[3,0,0,0]` would be transformed to
`[0,1,1,1]`.

The answer is then how many such redistribution cycles we require before we end
up at a configuration we have seen before (which when thinking about it I
concluded has to happen eventually).

My code is as follows:

{% highlight elixir %}
defmodule Aoc.Day6.Part1 do
  def steps_until_repeat(banks_string) when is_binary(banks_string) do
    banks_string |> String.trim |> String.split |> Enum.map(&String.to_integer/1) |> steps_until_repeat
  end
  def steps_until_repeat(banks), do: steps_until_repeat(banks, 1, MapSet.new |> MapSet.put(banks))
  def steps_until_repeat(banks, step, old_configs) do
    new_config = banks |> step()

    if MapSet.member?(old_configs, new_config) do
      step
    else
      steps_until_repeat(new_config, step + 1, MapSet.put(old_configs, new_config))
    end
  end

  def step(banks) do
    {max, max_bank} = banks |> Enum.with_index |> Enum.max_by(&(elem(&1, 0)))
    next_bank = rem(max_bank + 1, length(banks))
    cleared = List.replace_at(banks, max_bank, 0)
    add_ones(cleared, next_bank, max)
  end

  @doc """
  Adds 1 to each element from index `from` and num times

  ## Example

  iex> Part1.add_ones([0, 1, 2, 3, 4, 5], 4, 3)
  [1, 1, 2, 3, 5, 6]
  iex> Part1.add_ones([0, 1, 2, 4], 0, 14)
  [4, 5, 5, 7]
  iex> Part1.add_ones([1, 2, 3, 1], 0, 1)
  [2, 2, 3, 1]
  iex> Part1.add_ones([0, 1, 2, 0], 0, 5)
  [2, 2, 3, 1]

  """
  def add_ones(banks, from, num) do
    left = length(banks) - rem(from + num, length(banks))
    {rest, banks} = Enum.split(banks, from)
    {tail, neck} = inner_add_ones(banks, rest, num) |> Enum.split(left)
    neck ++ tail
  end
  def inner_add_ones(neck, tail, 0), do: neck ++ tail
  def inner_add_ones([], rest, num), do: inner_add_ones(rest, [], num)
  def inner_add_ones([head | tail], rest, num), do: inner_add_ones(tail, rest ++ [head + 1], num - 1)
end
{% endhighlight %}

I just simulate it exactly as described and keep track of old configurations
using a `MapSet`. The only slightly tricky thing is making sure that the list is
treated as a circular list, so that you restart at `0` when you reach the end.

I also wrote a recursive function to redistribute the ones, which splits the
list and then shuffles elements around. Not efficient but exactly the sort of
silly thing I like doing in these kinds of problems.

# Part 2
In part 2 we are are not asked how many we steps we need to start repeating
ourselves, but instead how many steps there are in the infinite loop in which we
repeat ourselves; difference being that from an initial configuration we take a
few steps that will never repeat, before we start the loop where we will repeat
ourselves infinitely.

Instead of using a `MapSet` as in part 1 I used a `Map` to keep track of
previous configurations and in which step these were seen. The first time we
repeat ourselves we can then just check which step we are at now and which one
we were are the last time.

{% highlight elixir %}
defmodule Aoc.Day6.Part2 do
  alias Aoc.Day6.Part1

  def cycle_length(banks_string) when is_binary(banks_string) do
    banks_string |> String.trim |> String.split |> Enum.map(&String.to_integer/1) |> cycle_length
  end
  def cycle_length(banks), do: cycle_length(banks, 1, %{banks => 0})
  def cycle_length(banks, step, old_configs) do
    new_config = banks |> Part1.step()

    if Map.has_key?(old_configs, new_config) do
      step - Map.get(old_configs, new_config)
    else
      cycle_length(new_config, step + 1, Map.put(old_configs, new_config, step))
    end
  end
end
{% endhighlight %}

# Final thoughts
This problem was not to difficult but it was quite fun. There is some repetition
of code between the two parts and I will possibly merge the two solutions into
one that answers both questions simultaneously. 
