---
layout: post
title:  "Day 22: Sporifica Virus"
date:   2017-12-22 12:00:00 +0100
categories:
tags: easy revisit
---
# Part 1
Today we are following a virus which walks around an infinite grid and infects
or disinfects cells. We start at the origin facing upwards and every time we
visit an infected cell we turn left and otherwise we turn right.

The code is as follows:

{% highlight elixir %}{% raw %}
defmodule Aoc.Day22.Part1 do
  def solve(input, steps) do
    input |> parse |> walk_stream |> Enum.at(steps) |> elem(3)
  end

  def parse(input) do
    lines = input |> String.split("\n")
    coords = lines |> Enum.with_index |> Enum.flat_map(&parse_row/1)
             |> Enum.into(MapSet.new())
    {coords, lines |> length}
  end

  def parse_row(row_and_y, x \\ 0, acc \\ [])
  def parse_row({"", _}, _, acc), do: acc
  def parse_row({"#" <> rest, y}, x, acc), do: parse_row({rest,y}, x+1, [{x,y} | acc])
  def parse_row({"." <> rest, y}, x, acc), do: parse_row({rest,y}, x+1, acc)

  def walk_stream({infected_nodes, width}) do
    initial = {starting_position(width), infected_nodes, {0, -1}, 0}
    Stream.iterate(initial, &iterate/1)
  end

  def iterate({{x,y}, infected_nodes, {xd, yd}, infect_bursts}) do
    infected = MapSet.member?(infected_nodes, {x,y})
   
    {infected_nodes, infect_bursts} = case infected do
      true -> {MapSet.delete(infected_nodes, {x,y}), infect_bursts}
      false -> {MapSet.put(infected_nodes, {x,y}), infect_bursts + 1}
    end

    {xd, yd} = case infected do
      true -> {-yd, xd} # Turn left
      false -> {yd, -xd} # Turn right
    end

    {x, y} = {x+xd, y+yd}

    {{x,y}, infected_nodes, {xd, yd}, infect_bursts}
  end

  def starting_position(width), do: {div(width-1, 2), div(width-1, 2)}
end
{% endraw %}{% endhighlight %}

There's nothing too complicated. I chose to use the `{x,y}`-tuple-keyed map once
again, which worked well; because the grid can be seen as growing as it is being
discovered a list-based grid will need to grow, which would require updating all
rows.

I was caught out by my axes at first; the parsing naturally causes the y axis to
be facing downward but my intuition from calculus has the y axis pointing up.
Once I noticed that everything worked the first time.

# Part 2
In part 2 the rules are slightly more complicated; the cells are in one of 4
states and the virus cycles between them. Depending on which state a cell is in
we update our direction in one of the four possible ways.

{% highlight elixir %}{% raw %}
defmodule Aoc.Day22.Part2 do
  @states [:clean, :weakened, :infected, :flagged, :clean]

  import Aoc.Day22.Part1, only: [parse_row: 1, starting_position: 1]

  def solve(input, steps) do
    input |> parse |> walk_stream |> Enum.at(steps) |> elem(3)
  end

  def parse(input) do
    lines = input |> String.split("\n")
    coords = lines |> Enum.with_index |> Enum.flat_map(&parse_row/1)
             |> Enum.map(fn({x,y}) -> {{x,y},:infected} end) |> Enum.into(%{})
    {coords, lines |> length}
  end

  def walk_stream({infected_nodes, width}) do
    initial = {starting_position(width), infected_nodes, {0, -1}, 0}
    Stream.iterate(initial, &iterate/1)
  end

  def iterate({{x,y}, infected_nodes, {xd, yd}, infect_bursts}) do
    state_before = Map.get(infected_nodes, {x,y}, :clean)
    state = next_state(state_before)
    infected_nodes = Map.put(infected_nodes, {x,y}, state)
   
    infect_bursts = case state do
      :infected -> infect_bursts + 1
      _ -> infect_bursts
    end

    {xd, yd} = case state_before do
      :clean -> {yd, -xd} # Turn right
      :weakened -> {xd, yd} # Keep going
      :infected -> {-yd, xd} # Turn left
      :flagged -> {-xd, -yd} # Turn around
    end

    {x, y} = {x+xd, y+yd}

    {{x,y}, infected_nodes, {xd, yd}, infect_bursts}
  end

  def next_state(current) do
    i = Enum.find_index(@states, &(&1 == current)) |> Kernel.+(1)
    Enum.at(@states, i)
  end
end
{% endraw %}{% endhighlight %}

# Final thoughts
I quite liked this one. It was not as difficult as the previous days but still
enough of a challenge to be worthwile.

This code is not too performant; solving part 2 takes about 10-15 seconds. I
might revisit this later to find out if it has to take that long.
