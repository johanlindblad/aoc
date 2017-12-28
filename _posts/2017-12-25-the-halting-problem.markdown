---
layout: post
title:  "Day 25: The Halting Problem"
date:   2017-12-25 12:00:00 +0100
categories:
tags: easy
---
# Part 1
The last task for this year is to simulate a [Turing
machine](https://en.wikipedia.org/wiki/Turing_machine). The puzzle input
consists of the program to run (i.e. the different states and how they write and 
branch depending on the value they read).

The question is how many times the number `1` appears on the tape.

My solution is as follows:

{% highlight elixir %}{% raw %}
defmodule Aoc.Day25.Part1 do
  def parse(input) do
    input
    |> String.trim
    |> String.split("\n")
    |> Enum.map(&(String.trim_trailing(&1, ":")))
    |> Enum.map(&(String.trim_trailing(&1, ".")))
    |> Enum.map(&String.trim_leading/1)
    |> parse_lines(%{})
  end

  def diagnostic_checksum(instructions) do
    simulate(instructions) |> Enum.at(instructions.steps) |> elem(4)
  end

  def simulate(instructions) do
    Stream.iterate({instructions.state, [], [], instructions, 0}, &step/1)
  end

  def step({state, [front | front_rest], [back | back_rest], instructions, num_ones}) do
    {write_val, dir, next_state} = instructions[state] |> elem(front)
    diff = write_val - front

    {tape_back, tape_front} = case dir do
      "right" -> {[write_val, back | back_rest], front_rest}
      "left" -> {back_rest, [back, write_val | front_rest]}
    end

    {next_state, tape_front, tape_back, instructions, num_ones + diff}
  end

  # Feed more of the infinite tape when needed...
  def step({state, [], tape_back, instructions, num_ones}) do
    step({state, [0], tape_back, instructions, num_ones})
  end
  def step({state, tape_front, [], instructions, num_ones}) do
    step({state, tape_front, [0], instructions, num_ones})
  end

  def parse_lines([], params), do: params
  def parse_lines(["" | tail], params), do: parse_lines(tail, params)

  def parse_lines(["Begin in state " <> s | tail], %{}) do
    parse_lines(tail, %{state: s})
  end

  def parse_lines(["Perform a diagnostic checksum after " <> rest | tail], params = %{}) do
    num = String.split(rest, " ") |> List.first |> String.to_integer
    parse_lines(tail, Map.put(params, :steps, num))
  end

  def parse_lines(["In state " <> state | tail], params) do
    {if_zero, tail} = parse_in_state(tail)
    {if_one, tail} = parse_in_state(tail)
    params = Map.put(params, state, {if_zero, if_one})

    parse_lines(tail, params)
  end

  def parse_in_state([
    "If the current value is " <> _,
    "- Write the value " <> val,
    "- Move one slot to the " <> dir,
    "- Continue with state " <> state | tail]) do
    {{String.to_integer(val), dir, state}, tail}
  end
end
{% endraw %}{% endhighlight %}

A majority of the code is quite explicit parsing of the instructions; they could
have been parsed in much fewer lines if we wanted to.

The actual simulation code creates a simulation stream, as I did many of the
previous days. The state consists of:
* The current state (for example, "A")
* The front, i.e. the part of the tape that is at or ahead of the writer. The
first element is the closest one.
* The back, i.e. the part of the tape that is behind the writer. The first
element is the closest one.
* The parsed instructions (these could be lifted out as they are constant)
* The number of ones written so far

Using a `front` and `rear` list is a pattern I have used earlier (such as with
the program in the virtual machine) and I find that it works well. In this case
we are just shuffling values from right at the writer to right in front or right
in the rear of it, so we are always taking the top element. This will be much
faster than replacing values further into the list (such as if we were to treat
the tape like a vector in languages that have them).

The iteration is performed by `step/1`, which simply does exactly as the
instructions say and rewind or forward the tape as appropriate. The last two
versions of `step/1` simply feed more tape whenever we run out of the length we
have generated.

# Part 2
There was no part 2 for today.

# Final thoughts
This was a fun puzzle and a good ending to a great Advent of Code. I find that I
have found some good patterns for doing these kinds of problems in a functional
way and the use of `front` and `rear` lists is one of them. It made this problem
very simple.
