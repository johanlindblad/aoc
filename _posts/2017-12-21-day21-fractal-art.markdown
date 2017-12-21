---
layout: post
title:  "Day 21: Fractal Art"
date:   2017-12-21 12:00:00 +0100
categories:
tags: medium revisit
---
# Part 1
Today we are doing a "Game of Life"-like thing where bits in a grid are turned
on and off. However, this problem is different in two ways:
1. The grid grows at each step
2. Parts of the grid are updated according to a lookup table

Essentially the grid is divided into squares of 3x3 (if the width is divisable
by 3) or 2x2 and are transformed by looking up their pattern to generate a new
4x4 or 3x3 square respectively. If no exact match is found we rotate and/or flip the square until we find one.

The question is how many bits are "on" after a number of iterations.

For the last problem involving grids I used a map with coordinate keys, which I
was very happy with. This time however I felt that this could be solved easier
with list operations and in the end I think I was right.

Basically the grid can be divided into 3x3 or 2x2 sections by a suitable
combination of chunking and transposing. That part of the code is a bit messy
but once it's written it is not likely to be rewritten.

The complete code is as follows:

{% highlight elixir %}
defmodule Aoc.Day21.Part1 do
  @initial [[0,1,0],[0,0,1],[1,1,1]]

  def parse_transformations(input) do
    String.trim(input) |> String.split("\n") |> Enum.map(&parse_row/1)
    |> Enum.into(%{})
    |> add_variations
  end
  def parse_row(row) do
    String.split(row, " => ") |> Enum.map(&parse_pattern/1) |> List.to_tuple
  end

  def on_after(patterns, steps) do
    grid_stream(patterns) |> Enum.at(steps) |> elem(0) |> num_squares
  end

  def num_squares(grid) do
    grid |> Enum.map(&Enum.sum/1) |> Enum.sum
  end

  def grid_stream({grid, width} \\ {@initial, 3}, patterns) do
    Stream.iterate({grid, width}, fn({grid, width}) ->
      new_grid = iterate({grid, width}, patterns)
      new_width = new_grid |> List.first |> length
      {new_grid, new_width}
    end)
  end

  def iterate({grid, width}, patterns) do
    cond do
      rem(width, 2) == 0 -> iterate({grid, width}, patterns, 2)
      rem(width, 3) == 0 -> iterate({grid, width}, patterns, 3)
    end
  end

  def iterate({grid, width}, patterns, chunk_size) do
    per_row = div(width, chunk_size)
    chunks = grid |> grid_to_chunks(chunk_size)
    transformed = chunks |> Enum.flat_map(&(Map.fetch!(patterns, &1)))

    # Chunk the list of squares into chunks of 3 or 4 rows (which is the number
    # of rows in one transformation), take as many of these as there will be per
    # row of output and then transpose and flatten these to get the new output
    # grid
    transformed |> Enum.chunk_every(chunk_size+1) |> Enum.chunk_every(per_row) |> Enum.flat_map(&transpose/1) |> Enum.map(&List.flatten/1)
  end

  # Gives a list of chunks, which need to be mapped and then re-chunked to
  # restore the grid
  def grid_to_chunks(grid, chunk_width) do
    grid |> Enum.chunk_every(chunk_width) |> Enum.flat_map(&(chunk_row_chunk(&1, chunk_width)))
  end
  def chunk_row_chunk(row_chunk, chunk_width) do
    row_chunk |> Enum.map(&(Enum.chunk_every(&1, chunk_width))) |> transpose
  end

  def all_variations(pattern) do
    Stream.cycle([:transpose, :rotate]) |> Enum.take(8)
    |> Enum.reduce([pattern], fn
      (:transpose, [head | tail]) -> [transpose(head), head | tail]
      (:rotate, [head | tail]) -> [Enum.reverse(head), head | tail]
    end)
  end

  def add_variations(patterns) do
    Enum.reduce(Map.keys(patterns), patterns, fn(from, patterns) ->
      to = patterns[from]
      Enum.reduce(all_variations(from), patterns, fn(pattern, patterns) ->
        Map.put(patterns, pattern, to)
      end)
    end)
  end

  def parse_pattern(coordinates \\ [[]], pattern, row \\ 0, column \\ 0)
  def parse_pattern([current | tail], <<char>> <> pattern, row, column) do
    coordinates = case <<char>> do
      "." -> [current ++ [0] | tail]
      "#" -> [current ++ [1] | tail]
      "/" -> [[], current | tail]
    end

    parse_pattern(coordinates, pattern, row, column)
  end
  def parse_pattern(grid, "", _, _), do: Enum.reverse(grid)

  def transpose(list), do: list |> List.zip |> Enum.map(&Tuple.to_list/1)
end
{% endhighlight %}

Basically we divide the grid into correct size chunks, map them using the lookup
table and then put the whole grid back together. Notice also that I generated
all rotations and transpositions on the pattern table instead of on each
generation. This made things a bit quicker.

# Part 2
For the first time, part 2 actually contains any new code. We are simply asked
for the same thing for a larger number of iterations.

# Final thoughts
As usual, I went for a straight-forward simulation solution the first time
around. There is a better solution which I have been planning in my head a bit.

My first thought was that every 3x3 grid generates 4 new 2x2 grids, which could
be handled separately in a recursive manner and likely also cached. This is
however complicated by the fact that when there are 3 2x2 grids in a row they
should be grouped into 2 3x3 grids instead (because 6 is divisable by 3).

This means that we need to handle that special case in order to have the more
elegant solution. I will likely revisit this problem to implement that.

Other than that, I am quite happy with this solution. The list operations are a
bit difficult to visualize but the rest of the code is not too complicated.
