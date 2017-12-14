---
layout: post
title:  "Day 2: Corruption Checksum"
date:   2017-12-02 12:00:00 +0100
categories:
tags: easy
---
# Part 1

Today we are given a problem involving calculating a checksum. It just consists
of finding the difference between the maximum and the minimum element in each
row and then summing these for every row.

(The full description can be found at
[http://adventofcode.com/2017/day/2](http://adventofcode.com/2017/day/2))

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day2.Part1 do
  def checksum(spreadsheet) do
    String.trim(spreadsheet)
    |> String.split("\n")
    |> Enum.map(&line_to_numbers/1)
    |> Enum.map(&delta/1)
    |> Enum.sum
  end

  def line_to_numbers(line) do
    String.split(line)
    |> Enum.map(&String.to_integer/1)
  end

  def delta(numbers) do
    Enum.max(numbers) - Enum.min(numbers)
  end
end
{% endhighlight %}

Essentially it just transforms the input to the required format and uses the
`Enum` module to calculate the max and the min.

## Part 2
For part 2, we are instead asked to find the two numbers that can be evenly
divided and produce the sum of these for each row.

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day2.Part2 do
  alias Aoc.Day2.Part1
  defdelegate line_to_numbers(line), to: Part1

  def checksum(spreadsheet) do
    String.trim(spreadsheet)
    |> String.split("\n")
    |> Enum.map(&line_to_numbers/1)
    |> Enum.map(&even_division/1)
    |> Enum.sum
  end

  defp even_division([_first]), do: false
  defp even_division([first, second | tail]) do
    cond do
      rem(first, second) == 0 -> div(first, second)
      rem(second, first) == 0 -> div(second, first)
      true -> even_division([first | tail]) || even_division([second | tail])
    end
  end
end
{% endhighlight %}

The finding of the two evenly dividing numbers is done in a recursive manner.
For the list `[a,b,c,d]` we first check if `a % b == 0` (b divides a) or the
opposite. If neither of these are true we check again for the list `[a,c,d]` as
well as `[b,c,d]`.

This is not as efficient as can be, seeing as both of these calls could later
check `[c,d]` but it is sufficient for our needs.

# Final thoughts
I am happy enough with this solution. Because our input is small, there is no
need to further optimize anything, and writing the recursive version was more
fun than doing it the correct way.
