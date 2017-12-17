---
layout: post
title:  "Day 15: Dueling Generators"
date:   2017-12-15 12:00:00 +0100
categories:
tags: easy
---
# Part 1
Today we are working with what turns out are two fairly standard pseudo-random 
number generators. We are asked how many times they have equal lower 16 bits
when generating numbers a few million times.

I wrote my solution with streams:

{% highlight elixir %}
defmodule Aoc.Day15.Part1 do
  use Bitwise

  def parse(file_input), do: file_input |> String.trim |> String.split("\n") |> Enum.map(&parse_row/1)
  def parse_row("Generator " <> <<_::binary-size(1)>> <> " starts with " <> number) do
    String.to_integer(number)
  end

  def solve(input = [_a_first, _b_first]), do: number_streams(input) |> num_equal

  def number_streams([a_first, b_first]) do
    {number_stream(a_first, 16807, 2147483647), number_stream(b_first, 48271, 2147483647)}
  end

  def num_equal({a_stream, b_stream}, pairs \\ 40_000_000) do
    Stream.zip(a_stream, b_stream)
    |> Stream.take(pairs)
    |> Stream.filter(fn({a, b}) -> matches_last_16_bits?(a, b) end)
    |> Enum.count
  end

  def number_stream(first_value, factor, modulo) do
    Stream.iterate(first_value, &(next_value(&1, factor, modulo)))
    |> Stream.drop(1)
  end

  def next_value(last_value, factor, modulo) do
    rem(last_value * factor, modulo)
  end

  def matches_last_16_bits?(number_a, number_b) do
    lower_16_bits(number_a) == lower_16_bits(number_b)
  end

  defp lower_16_bits(number) do
    number &&& (65535)
  end
end
{% endhighlight %}

It can be shortened a bit but otherwise it is pretty much exactly as specified
by the problem description.

# Part 2
In part 2 we are interested only in the numbers that can be evenly divided by 4
and 8 (for the two generators respectively). My solution just builds slightly on
part 1:

{% highlight elixir %}
defmodule Aoc.Day15.Part2 do
  use Bitwise
  alias Aoc.Day15.Part1

  def even_division(stream, modulo) do
    stream |> Stream.filter(&(rem(&1, modulo) == 0))
  end

  def solve(input = [_a, _b]) do
    {a, b} = Part1.number_streams(input)
    Part1.num_equal({a |> even_division(4), b |> even_division(8)}, 5_000_000)
  end
end
{% endhighlight %}

# Final thoughts
This was a quite simple problem. Because the generators described are standard
RNGs (at least in C++) they have likely gone through at least some
cryptographical scrutiny; it is unlikely that this problem can be solved by
other means than actually doing all the iterations.
