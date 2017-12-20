---
layout: post
title:  "Day 19: A Series of Tubes"
date:   2017-12-18 12:00:00 +0100
categories:
tags: medium revisit
---
# Part 1
Today actually begins with a video:

{% raw %}
<iframe width="560" height="315" src="https://www.youtube.com/embed/QdCBdRxXbDk" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>
{% endraw %}
(more on this later)

As can be seen (despite the terrible video capture quality), we are traversing a
series of tubes. We dash through a set of pipes (excuse the pun), which is
represented by `|` and `-` for straight sections and `+` for corners. In
addition, there are letters placed along the way and we are interested in
finding out which these are and in what order.

Once again we are at a grid-based puzzle (as in day 14). I was unhappy with my
first solution for day 14, in which I represented the grid as lists of lists.
After some research, I found that most seem to prefer doing grids as maps with
tuple keys in Elixir and so I decided to give it a try. I found it to really
work well.

The only problem is that it is not immediately known what the dimensions of the
grid are so those have to be stored elsewhere if they are needed (I could just
rely on maps returning `nil` for undefined keys where needed).

Of course, maps will not be as performant as arrays of contiguous memory as you
have in C++ and the like but they worked well here.

Here is the code:

{% highlight elixir %}{% raw %}
defmodule Aoc.Day19.Part1 do
  def parse(input) do
    lines = input |> String.split("\n")
    lines |> Enum.with_index |> Enum.map(&parse_row/1) |> List.flatten |> Enum.into(%{})
  end
  def parse_row({row, y}) do
    chars = row |> String.graphemes
    chars |> Enum.with_index |> Enum.map(fn({char, x}) -> {{x, y}, char} end)
  end

  def collected(table) do
    table
    |> coordinate_stream
    |> Stream.map(fn({x, y}) -> table[{x,y}] end)
    |> Stream.filter(fn(<<char>>) -> char in ?A..?Z end)
    |> Enum.join("")
  end

  def coordinate_stream(table), do: Stream.map(walk_stream(table), fn({coord, _}) -> coord end)

  def walk_stream(table) do
    Stream.iterate({start_index(table), {0, 1}}, fn({coords, direction}) ->
      walk(table, coords, direction)
    end)
    |> Stream.take_while(fn({_, direction}) -> direction != {0, 0} end)
  end

  def start_index(table = %{}, _candidate = {x,0} \\ {0, 0}) do
    case table[{x, 0}] do
      "|" -> {x, 0}
      _ -> start_index(table, {x + 1, 0})
    end
  end

  def walk(table, coordinate, direction \\ {0, 1})
  def walk(table = %{}, {x, y}, {xd, yd}) when x >= 0 and y >= 0 do
    {xd, yd} = cond do
      can_walk?(table[{x+xd,y+yd}]) -> {xd, yd}
      can_walk?(table[{x+yd,y+xd}]) -> {yd, xd}
      can_walk?(table[{x-yd,y-xd}]) -> {-yd, -xd}
      true -> {0, 0}
    end

    {{x+xd, y+yd}, {xd, yd}}
  end

  def can_walk?( <<char>>) when char in ?A..?Z, do: true
  def can_walk?(pipe) when pipe in ["|", "-", "+"], do: true
  def can_walk?(_), do: false
{% endraw %}{% endhighlight %}

I am quite happy with the solution. After parsing the grid into the map, we find
the starting point by just traversing the first row (as described) and finding
the pipe.

Then, we can just use `Stream.iterate/1` to start from this coordinate and use
`walk/3` to generate the next coordinate. `walk/3` first checks if we can
walk to the next square in the same direction and if not, it attempts to turn 90
degrees to the right or to the left.

If we cannot do any of these things we must stop, as we are not allowed to go
back in the same direction. This terminates the stream.

We then use this stream in `coordinate_stream/1`, which extracts the traversed
from the rest of the state (which also contains the direction) and then use
`collected/1` to simply map these coordinates to their respective characters and
keep all the letters.

# Part 2
Today's part 2 was really simple (if you solved it right). For me the solution
consists of 1 line of actual code:

{% highlight elixir %}
defmodule Aoc.Day19.Part2 do
  def num_steps(simulation_stream) do
    simulation_stream |> Enum.count
  end
end
{% endhighlight %}

We just need to see how many steps we are taking.

# Final thoughts
I really liked this problem. It was a nice excuse to revisit the question of how
to do 2D grids in a functional way and in the end the cordinate tuple map worked
well for my needs.

Once again I created a stream-based solution to allow me to easily inspect every
step of the way. This made it really easy to create nice visualizations of the
path taken.

Left out from the part 1 code were the two functions `visualization_stream/2`
and `filled_input/1` which provide two visualization over the coordinate stream.
They can be found on (Github)[https://raw.githubusercontent.com/johanlindblad/aoc-2017/master/lib/aoc/day19/part1.ex]
* `visualization_stream/1` plays the path through the pipes with a moving
viewport. It can be run in `iex -S mix` with `Aoc.puzzle_input(19) |>
Aoc.Day19.Part1.parse |> Aoc.Day19.Part1.visualization_stream |> Stream.run`
* `filled_input/1` instead prints the entire grid and fills in squares in the
path taken. It can be run in the same manner but requires a really large
terminal with a small font.
