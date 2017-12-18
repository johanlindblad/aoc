---
layout: post
title:  "Day 3: Spiral Memory"
date:   2017-12-03 12:00:00 +0100
categories:
tags: hard
---
# Part 1

This was definitely a fun one, which I have revisited twice and spent a fair
amount of time on. The problem describes a memory system where cells are
arranged in a counter-clockwise spiral (in a square grid). We are concerned with
the [Manhattan distance](https://en.wiktionary.org/wiki/Manhattan_distance) from 
the center to a specific square.

My experience from earlier years told me that we would probably need to simulate
the spiral for part 2 so I chose to do precisely that. I did however notice some
mathematical connections when I first attempted this problem:

* The layer widths are [triangular numbers](https://en.wikipedia.org/wiki/Triangular_number)
* The lower-right corner of each layer are perfect squares of odd numbers

Because I build the spiral layer-by-layer I did however not need these (the
width just increases by 2).

I decided to use this problem to learn more about Elixir's `Stream` module. If
we keep track of a few variables we can generate the next square from the
previous one. This means that the whole grid can be seen as an infinite stream
that we can take squares from until our patience runs out (or erlang's number
type).

The solution for part 1 is as follows:

{% highlight elixir %}
defmodule Aoc.Day3.Part1 do
  require Record
  Record.defrecord :square, address: 1, layer: 1, layer_width: 1, from_start: 0, to_next_layer: 1, from_corner: 0, to_corner: 0

  def squares, do: Stream.iterate(square(), &next_square_after/1)
  def steps(square), do: squares() |> Enum.at(square - 1) |> distance

  def next_square_after(square(address: 1)), do: {:square, 2, 2, 2, 0, 8, 1, 1}
  def next_square_after(sq = square(to_next_layer: 1, layer: layer, layer_width: layer_width)) do
    increment(sq) |> square(
      layer: layer + 1, layer_width: layer_width + 2,
      from_start: 0, to_next_layer: (layer_width + 2) + 4,
      from_corner: 1, to_corner: layer_width + 1
    )
  end
  def next_square_after(sq = square()), do: increment(sq)

  def increment({:square, address, layer, layer_width, from_start, to_next_layer, from_corner, _}) do
    from_corner = rem(from_corner + 1, layer_width)
    {:square, address + 1, layer, layer_width, from_start + 1, to_next_layer - 1, from_corner, layer_width - from_corner}
  end

  def distance(square(layer: layer, from_corner: from_corner, to_corner: to_corner)) do
    abs(from_corner - to_corner) |> div(2) |> Kernel.+(layer - 1)
  end
end
{% endhighlight %}

I chose to use the `Record` module to pass around a more proper bag of variables
than just a tuple. The convenience functions and pattern matching this provides
makes the code much nicer in my mind.

The `squares/0` function starts the stream. We just start at 1 and generate the
next square from the previous one with the help of the `next_square_after/1`
function (which usually just calls the `increment/1` function).

Generating the next square is in most cases very simple. We just need to:
* Increment the address
* Indicate that we are one step further from the start of this layer...
* ...as well as one step closer to the start of the next
* Indicate that we are one step closer to the next corner...
* ...as well as one step further from the last one

The only exceptions are:
* The corner intervals cycle (in cycles of `layer_width`) so we use a modulo
operation
* When we are 1 step from the start of the next layer, we need to increase layer
number (by 1), the layer width (which increases by 2) and the distance to the
next layer (which is just the new layer width times 4)

When all of this is done we need to calculate the manhattan distance. This
distance consists of two parts:
* The distance from the center of the spiral straight out to the middle of the
current row or column. This distance is just 1 less than the layer number.
* The distance from the middle of the current row or column straight to the
square we are interested in. If we think about this we notice that this is just
the average of the distance to the next corner and the last one.

## Part 2
In part two we are interested in the values of squares. They are filled from
address 1 upwards and are the sum of the adjacent squares that are already
filled out. This is where I started to at least consider that I should not have
"untied" the spiral in the way that I have but instead cared more about
coordinates. Oh well.

After some time with paper and pen I came to the conclusion that we can model
the neighbours as three different types:

1. The adjacent ones, which are the previous one and, if we are right after a
corner, the one previous to that one as well.
2. The inner ones, which are right inwards as well as possibly right before or
after that one (depending on if we are at or next to a corner)
3. The ones from the start of the same layer (right when we wrap around).

These are calculated as relative offsets (i.e. difference between this square
number and the previous ones) by the functions  `adjacent_offsets/1`,
`inner_offsets/1` and `start_of_layer_offsets/1` respectively. These rely
heavily on pattern matching to handle all different special cases.

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day3.Part2 do
  alias Aoc.Day3.Part1
  import Aoc.Day3.Part1
  use Bitwise

  def relative_indices, do: Part1.squares |> Stream.map(&relative_indices/1)
  def values, do: relative_indices() |> Stream.transform([], &value_accumulator/2)
  def first_greater_than(n), do: values() |> Stream.drop_while(&(&1 < n)) |> Enum.at(0)

  # Emits {[emitted elements], [accumulated previous squares]}
  def value_accumulator(_, []), do: {[1], [1]}
  def value_accumulator(offsets, previous_values) do
    offsets = Enum.map(offsets, &(-&1 - 1))
    val = sum_of(previous_values, offsets)
    {[val], [val | previous_values]}
  end

  # Hard-code first three so that we are guaranteed unique indices
  def relative_indices(square(address: 1)), do: []
  def relative_indices(square(address: 2)), do: [-1]
  def relative_indices(square(address: 3)), do: [-1, -2]
  def relative_indices(sq = square()) do
    adjacent_offsets(sq) ++ start_of_layer_offset(sq) ++ inner_offsets(sq)
  end

  def adjacent_offsets(square(from_start: 0)), do: [-1]
  def adjacent_offsets(square(from_corner: 1)), do: [-1, -2]
  def adjacent_offsets(square(from_start: 1)), do: [-1, -2]
  def adjacent_offsets(_), do: [-1]

  def inner_offsets(sq = square(from_corner: 0)), do: [inner_offset(sq)]
  def inner_offsets(sq = square(layer: 2)), do: [inner_offset(sq)]
  def inner_offsets(sq = square(from_start: 0)), do: [inner_offset(sq)]
  def inner_offsets(sq = square(from_start: 1)), do: [inner_offset(sq), inner_offset(sq) - 1]
  def inner_offsets(sq = square(to_corner: 1)), do: [inner_offset(sq), inner_offset(sq) - 1]
  def inner_offsets(sq = square(from_corner: 1)), do: [inner_offset(sq) + 1, inner_offset(sq)]
  def inner_offsets(sq), do: [inner_offset(sq) + 1, inner_offset(sq), inner_offset(sq) - 1]

  def start_of_layer_offset(square(to_next_layer: 1, from_start: index)), do: [-index]
  def start_of_layer_offset(square(to_next_layer: 2, from_start: index)), do: [-index]
  def start_of_layer_offset(_), do: []

   def inner_offset(sq = square()) do
    -(4 * inner_layer_width(sq)) - (2 * corners_passed(sq)) - inner_offset_extra(sq)
  end
  defp inner_offset_extra(square(from_corner: 0)), do: 2
  defp inner_offset_extra(square(from_start: 0)), do: 0
  defp inner_offset_extra(square(from_start: 1)), do: 0
  defp inner_offset_extra(_), do: 1

  def corners_passed(square(from_start: index, layer_width: width)), do: div(index, width)
  def inner_layer_width(square(layer_width: width)), do: width - 2

  def sum_of(elements, indices, steps \\ 0)
  def sum_of([head | tail], [steps | indices], steps), do: head + sum_of(tail, indices, steps + 1)
  def sum_of(_, [], _), do: 0
  def sum_of([_ | tail], indices, steps), do: sum_of(tail, indices, steps + 1)
end
{% endhighlight %}

With these calculated I once again build a `Stream`, this time of these relative
indices for each square, which we can then use `Stream.transform/3` to calculate
with an accumulator for previous square values.

Lastly, I wrote `sum_of/3`, which sums specific elements from a list. This was
more a fun recursive exercise than what I would consider a nice solution.

# Final thoughts
This was a very fun problem where I got to both play with some math on pen and
paper and practice using Elixir's streams. It is probably the problem which I
spent most time on (including the revisit) but also the one where I am most
happy with the solution.
