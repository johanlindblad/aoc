---
layout: post
title:  "Day 12: Digital Plumber"
date:   2017-12-12 12:00:00 +0100
categories:
tags: medium
---
# Part 1
Today we are looking at a network of pipes (the internet, maybe?). 

{% highlight elixir %}
defmodule Aoc.Day11.Part2 do
  alias Aoc.Day11.Part1

  def solve(input), do: input |> Part1.preprocess |> max_distance

  def max_distance(steps) do
    steps
    |> Enum.map(&Part1.effect/1)
    |> Enum.reduce({0, {0, 0}}, fn(step, {max_d, coord}) ->
      coord = Part1.sum(coord, step)
      {max(max_d, Part1.distance(coord)), coord}
    end)
    |> elem(0)
  end
end
{% endhighlight %}

# Part 2

{% highlight elixir %}
defmodule Aoc.Day11.Part2 do
  alias Aoc.Day11.Part1

  def solve(input), do: input |> Part1.preprocess |> max_distance

  def max_distance(steps) do
    steps
    |> Enum.map(&Part1.effect/1)
    |> Enum.reduce({0, {0, 0}}, fn(step, {max_d, coord}) ->
      coord = Part1.sum(coord, step)
      {max(max_d, Part1.distance(coord)), coord}
    end)
    |> elem(0)
  end
end
{% endhighlight %}

# Final thoughts
This was a quite fun one. I am tempted to try it again with one of the other
ways of doing hexadecimal coordinates.
