---
layout: post
title:  "Day 17: Spinlock"
date:   2017-12-17 12:00:00 +0100
categories:
tags: medium revisit
---
# Part 1
Today we are adding numbers to lists with equally long jumps in between.
Starting from the empty list we keep adding increasing numbers and are the
interested in knowing which one is next to the last one we inserted.

This ends up being quite simple with the help of the `List` module:

{% highlight elixir %}
defmodule Aoc.Day17.Part1 do
  def value_after(num_insertions, steps) do
    {index, list} = Enum.reduce(0..num_insertions, {0, []}, fn(_i, {index, list}) ->
      step(list, index, steps)
    end)

    Enum.at(list, index + 1)
  end

  def step(list, current_index, steps)
  def step([], _, _), do: {1, [0]}
  def step(list, current_index, steps) do
    value = length(list)
    index = (current_index + steps) |> rem(length(list)) |> Kernel.+(1)
    {index, List.insert_at(list, index, value)}
  end
end
{% endhighlight %}

# Part 2
In part 2 we are again given the instructions to perform the same operation
several orders of magnitude more times. It is no longer feasible to simulate the
entire process.

Fortunately, we notice that after 0, no elements will ever be inserted at index
0. This follows from the fact in part 1 that:
* We calculate the index by adding the current index and the step and then
taking the remainder modulo the list size (because the list wraps around).
* We add 1 because we want to insert *after* this index.
* The remainder is in this case always nonnegative so it can not be less than 0,
meaning that when we add 1 we get no less than 1.

This means that the first value after 0 will be whichever one we insertd at
index 1 last. It is therefore sufficient to just iterate without building the
entire list, but instead just keep track of the largest number that was inserted
at index 1 so far.

This gives a quite short solution:

{% highlight elixir %}
defmodule Aoc.Day17.Part2 do
  def value_after(num_insertions, steps) do
    {last, _} = Enum.reduce(0..num_insertions, {0, 0}, fn(current, {last, index}) ->
      next_index = (1 + index + steps) |> rem(current + 1)

      case index do
        0 -> {current, next_index}
        _ -> {last, next_index}
      end
    end)

    last
  end
end
{% endhighlight %}

# Final thoughts
I like the flavor of the puzzles these past few days. The pattern seems to be to
do a part 1 where there is no obvious solution other than simulating and then
modifying the problem slightly in part 2 so that you can figure out a short-cut. 

I will likely revisit this later, as I have two ideas for improvement.

Firstly, it should be possible to make my simulation code faster in part 1 by
inserting several items in the same pass through the list. Once the list grows
larger than the skip size, we will insert numbers at increasing indices. With 
linked lists such as the ones we have in Elixir it should be significantly
faster to avoid traversing the list unnecessarily often.

Secondly, I think there could possibly be a solution similar to part 2 for the
first part too. We can calculate in advance where the last item will be inserted
(that is just some modular arithmetic) and so it should be possible to know in
advance which items will be inserted before or after the point this ends up
being. I just need to let this solution pan itself out in the back of my head
for a while.
