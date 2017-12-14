---
layout: post
title:  "Day 8: I Heard You Like Registers"
date:   2017-12-08 12:00:00 +0100
categories:
tags: easy
---
# Part 1
The puzzle for today asks us to parse instructions for a CPU. All instructions
are simple increments or decrements and they are executed conditionally based on
the value of a different (or the same) register. The only catch is that the
registers are named rather than numbered and we do not know in advance which
registers the code mentions.

At the end we ask what the largest value is in any of the registers.

My code is as follows:

{% highlight elixir %}
defmodule Aoc.Day8.Part1 do
  def parse(input) do
    String.trim(input) |> String.split("\n") |> Enum.map(&parse_row/1)
  end

  def parse_row(row), do: String.split(row) |> clean

  def simulate(instructions), do: simulate(instructions, %{})
  def simulate([], registers), do: Map.values(registers) |> Enum.max
  def simulate([instruction | rest], registers) do
    [register, operation, operand | _] = instruction

    registers = case check_cond(instruction, registers) do
      true -> registers_after(registers, register, operation, operand)
      false -> registers
    end

    simulate(rest, registers)
  end

  def registers_after(registers, reg, op, opa) do
    current = registers[reg] || 0
    Map.put(registers, reg, operation(current, op, opa))
  end

  def check_cond([_, _, _, reg, op, val], registers) do
    Map.get(registers, reg, 0) |> inner_cond(op, val)
  end
  def inner_cond(val, :>, other), do: val > other
  def inner_cond(val, :<, other), do: val < other
  def inner_cond(val, :>=, other), do: val >= other
  def inner_cond(val, :<=, other), do: val <= other
  def inner_cond(val, :==, other), do: val == other
  def inner_cond(val, :!=, other), do: val != other

  def operation(val, :inc, by), do: val + by
  def operation(val, :dec, by), do: val - by

  def clean([reg, op, opa, "if", con_reg, con_op, con_val]) do
    [String.to_atom(reg), String.to_atom(op), String.to_integer(opa), String.to_atom(con_reg), String.to_atom(con_op), String.to_integer(con_val)]
  end
end
{% endhighlight %}

I think my solution is fairly decent. I used a `Map` to keep track of the
registers and used pattern matching to make most of the simulation code quite
simple. 

# Part 2
Part 2 just slightly changes the question from "what is the largest register
value at the end?" to "what was the largest register value at any time during
the simulation?". The code was modified slightly to keep track of this:

{% highlight elixir %}
defmodule Aoc.Day8.Part2 do
  alias Aoc.Day8.Part1
  defdelegate parse(input), to: Part1

  def simulate(instructions), do: simulate(instructions, %{}, 0)
  def simulate([], _registers, max_so_far), do: max_so_far
  def simulate([instruction | rest], registers, max_so_far) do
    [register, operation, operand | _] = instruction

    registers_after = Part1.registers_after(registers, register, operation, operand)

    registers = case Part1.check_cond(instruction, registers) do
      true -> registers_after
      false -> registers
    end

    value = Map.get(registers, register, 0)

    simulate(rest, registers, max(max_so_far, value))
  end
end
{% endhighlight %}

# Final thoughts
I quite like these sorts of problems. There are much fewer instructions here
than in the Synacor challenge or some earlier problems in the AoC so it was much
less complex than it could be but still fun.

I could clean up the code slightly but I don't see anything in my approach that
I don't like.

