---
layout: post
title:  "Day 14: Disk Defragmentation"
date:   2017-12-14 12:00:00 +0100
categories:
tags: hard revisit
---
# Part 1
This problem concerns a disk that is supposed to be defragmented. What the
description boils down to is checking how many bits are set in a knot hash from
day 10.

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day14.Part1 do
  alias Aoc.Day10.Part2

  def num_ones(input) do
    input |> grid |> Enum.map(&Enum.sum/1) |> Enum.sum
  end

  def grid(input) do
    Enum.map(0..127, fn(i) ->
      input <> "-#{i}"
      |> Part2.run |> hex_to_bin
    end)
  end

  def hex_to_bin(input)
  def hex_to_bin(<<a>> <> rest) do
    {integer, ""} = Integer.parse(<<a>>, 16)
    bin = Integer.digits(integer, 2) |> lpad(4)
    bin ++ hex_to_bin(rest)
  end
  def hex_to_bin(<<>>), do: []

  def lpad(list, to) when length(list) == to, do: list
  def lpad(list, to), do: lpad([0 | list], to)

  def count_ones(bits), do: Enum.count(bits, fn(x) -> x end)
end
{% endhighlight %}

This part was pretty straight-forward. There is no need to actually build the
grid but as I expected (and as often happens), it was needed in part 2.

# Part 2
Part 2 was much more tricky. We are basically asking how many connected
components there are in the graph of set bits. I ended up writing a 
[union-find data structure](https://en.wikipedia.org/wiki/Disjoint-set_data_structure)
which was easier than when I was to write it the optimal way in C++ but which 
is also not as efficient as we do not have arrays we can access in constant time.

{% highlight elixir %}{% raw %}
defmodule Aoc.Day14.Part2 do
  alias Aoc.Day14.Part1

  @doc """
  TODO: flood-fill
  """
  def solve(input) do
    Part1.grid(input)
    |> fill_spots
    |> join_spots
    |> disjoint_set_num_sets
  end

  def fill_spots(grid) do
    {rows, _start_from} = 
      Enum.reduce(grid, {[], 1}, fn (row, {rows, start_from}) -> 
        {row, start_from} = fill_row_spots(row, start_from)
        {rows ++ [row], start_from}
      end)

    rows
  end

  def fill_row_spots(row, start_from) do
    Enum.reduce(row, {[], start_from}, fn
      (0, {spots, start_from}) -> {spots ++ [0], start_from}
      (1,{spots, start_from}) -> {spots ++ [start_from], start_from + 1}
    end)
  end

  def join_spots(filled_spots) do
    sets = to_disjoint_set(filled_spots)
    sets = join_adjacent(filled_spots, sets)
    sets = join_upper(filled_spots, sets)
    sets
  end

  def to_disjoint_set(filled_spots) do
    List.flatten(filled_spots)
    |> Enum.filter(&(&1 > 0))
    |> Enum.reduce(%{}, &(disjoint_set_insert(&2, &1)))
  end

  def join_adjacent(filled_spots, sets) do
    Enum.reduce(filled_spots, sets, fn(row, sets) ->
      Enum.chunk_by(row, &(&1 > 0))
      |> Enum.reduce(sets, fn
        ([0 | _zeros], sets) -> sets
        (indices, sets) -> disjoint_set_join(sets, indices)
      end)
    end)
  end

  def join_upper(filled_spots, sets) do
    {_, sets} = Enum.reduce(filled_spots, {[], sets}, fn
      (row, {[], sets}) -> {row, sets}
      (row, {last_row, sets}) ->
        sets = Enum.zip(row, last_row)
        |> Enum.reduce(sets, fn
          ({i,j}, sets) when i > 0 and j > 0 -> disjoint_set_join(sets, i, j)
          (_, sets) -> sets
        end)
        {row, sets}
    end)

    sets
  end

  def disjoint_set_insert(sets = %{}, number) do
    Map.put(sets, number, MapSet.new([number]))
  end
  def disjoint_set_join(sets = %{}, first, first), do: sets
  def disjoint_set_join(sets = %{}, first, second) do
    case {sets[first], sets[second]} do
      {set1 = %MapSet{}, set2 = %MapSet{}} ->
        sets |> Map.replace(first, MapSet.union(set1, set2)) |> Map.replace(second, first)
      {%MapSet{}, _root} ->
        disjoint_set_join(sets, second, first)
      {root, set = %MapSet{}} ->
        sets |> Map.replace(second, MapSet.put(set, first)) |> disjoint_set_join(root, second)
      {root1, root1} -> sets
      {root1, root2} ->
        sets |> disjoint_set_join(root1, root2)
    end
  end
  def disjoint_set_join(sets = %{}, [key | keys]) do
    Enum.reduce(keys, sets, &(disjoint_set_join(&2, key, &1)))
  end

  def disjoint_set_num_sets(sets = %{}) do
    Map.values(sets) |> Enum.reject(&is_integer/1) |> Enum.count
  end
end
{% endraw %}{% endhighlight %}

I build the grid and run two passes through it:
1. Joining adjacent nodes in the same row
2. Joining adjacent nodes in the same column

I do both of these passes with the union-find data structure, with each set bit
is counted as one member. It is not very efficient and there is way too much
code.

# Final thoughts
This was a fun problem which was made much more difficult by my choice of
language. I will definitely try to revisit this in the future and see how
difficult it is to implement (flood fill)[https://en.wikipedia.org/wiki/Flood_fill]
in a functional way (or possibly some other approach). Even if not, I wrote this
solution in a rush so there is cause for at least cleaning it up a bit.
