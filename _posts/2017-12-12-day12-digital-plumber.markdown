---
layout: post
title:  "Day 12: Digital Plumber"
date:   2017-12-12 12:00:00 +0100
categories:
tags: medium
---
# Part 1
Today we are looking at a network of pipes (the internet, maybe?). This consists
of program that can talk to other programs as long as there is a way between
them. Essentially then we are looking at a graph with possibly more than one
component. The first part wants us to check how many programs are in the same
component as the first one.

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day12.Part1 do
  def solve(input), do: input |> preprocess |> in_same_group |> elem(0)

  def preprocess(input) do
    String.trim(input) |> String.split("\n") |> Enum.map(&preprocess_row/1)
    |> Enum.into(%{})
  end
  def preprocess_row(row) do
    [program, "<->" | rest] = String.split(row)
    linked = rest |> Enum.map(&String.replace_trailing(&1, ",", "")) |> Enum.map(&String.to_integer/1)
    {String.to_integer(program), linked}
  end

  def in_same_group(graph, node \\ 0, visited \\ MapSet.new()) do
    unvisited = Enum.reject(graph[node], &(MapSet.member?(visited, &1)))
    visited = [node | unvisited] |> MapSet.new() |> MapSet.union(visited)

    {count, new_visited} = 
      unvisited 
      |> Enum.map(&(in_same_group(graph, &1, visited))) 
      |> Enum.reduce({0, MapSet.new()}, fn({count, visited}, {total, acc_visited}) ->
        {count + total, MapSet.union(visited, acc_visited)}
      end)

    {count + 1, MapSet.union(visited, new_visited)}
  end
end
{% endhighlight %}

It is a fairly simple breadth-first solution. We just parse the instructions and
follow any links as long as they are to nodes that are unvisited.

# Part 2
The next part is concerned with how many connected components there are. The
solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day12.Part2 do
  alias Aoc.Day12.Part1

  def solve(input), do: input |> Part1.preprocess |> num_groups |> elem(0)

  def num_groups(graph, visited \\ MapSet.new) do
    Enum.reduce(Map.keys(graph), {0, visited}, fn(node, {count, visited}) ->
      case MapSet.member?(visited, node) do
        true -> {count, visited}
        false -> 
          {_, new_visited} = Part1.in_same_group(graph, node, visited)
          {count + 1, MapSet.union(new_visited, visited)}
      end
    end)
  end
end
{% endhighlight %}

We use the function from part 1 and iterate over the set of nodes, keeping track
of which ones we have already visited.

# Final thoughts
This one was good practice on doing things with graphs in a functional way. I
don't think you can quite implement a efficient [union-find data structure](https://en.wikipedia.org/wiki/Disjoint-set_data_structure) in a functional way but if I had one written this problem could be solved easier.
