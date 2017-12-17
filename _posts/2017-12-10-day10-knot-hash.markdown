---
layout: post
title:  "Day 10: Knot Hash"
date:   2017-12-10 12:00:00 +0100
categories:
tags: medium
---
# Part 1
Today's challenge consists of calculating a hash. This is done by taking the
a list of the numbers from `0` to `255` and reversing certain sections based on
the input. This is done starting from a pointer which goes through the list with
increasing speed.

{% highlight elixir %}
defmodule Aoc.Day10.Part1 do
  def run(input_string, size \\ 256)
  def run(input_string, size) when is_binary(input_string) do
    input_string |> String.trim |> String.split(",") |> Enum.map(&String.to_integer/1)
    |> hash(size)
    |> elem(3)
  end

  def hash(instructions, size), do: hash(Enum.to_list(0..size-1), size, instructions)
  def hash(list, size, instructions, position \\ 0, skip_size \\ 0)
  def hash(list, _size, [], position, skip_size) do
    product = rotate(list, -position) |> Enum.take(2) |> Enum.reduce(&Kernel.*/2)
    {list, position, skip_size, product}
  end
  def hash(list, size, [instruction | rest], position, skip_size) do
    rotate_n = instruction + skip_size
    list = list
           |> Enum.reverse_slice(0, instruction)
           |> rotate(rotate_n)

    position = position + rotate_n
    hash(list, size, rest, position, skip_size + 1)
  end

  def rotate(list, steps)
  def rotate(list, 0), do: list
  def rotate(list, steps) when steps >= length(list), do: rotate(list, rem(steps, length(list)))
  def rotate(list, steps) when steps < 0, do: rotate(list, length(list) + rem(steps, length(list)))
  def rotate([head | tail], steps), do: rotate(tail ++ [head], steps - 1)
end
{% endhighlight %}

My solution consists of three parts:
1. An input parser, which is simple enough
2. The main hash function, which reverses certain parts and rotates the list
3. The recursive list rotation function

My implementation is a bit inefficient, mainly due to the use of `3.`. Because
appending to a linked list is `O(n)`, this takes quite a bit of time. This was
done to simplify other parts of the code (so that the slice to be reversed
always was at the beginning). In retrospect this did however cause some problems
in the hard-to-debug part 2.

# Part 2
In part 2 we are now treating the input instead as `ASCII` and performing
multiple rounds. The output is then to be `XOR`:ed in chunks of 16 and converted
to hexadecimal.

{% highlight elixir %}
defmodule Aoc.Day10.Part2 do
  alias Aoc.Day10.Part1
  use Bitwise

  def run(input_string) do
    instructions = input_string <> <<17, 31, 73, 47, 23>> |> :binary.bin_to_list
    rounds(Enum.to_list(0..255), instructions, 64)
    |> sparse_to_dense
    |> dense_to_hex
  end

  def rounds(bytes, instructions, rounds, position \\ 0, skip \\ 0)
  def rounds(bytes, _instructions, 0, position, _), do: Part1.rotate(bytes, -position)
  def rounds(bytes, instructions, rounds, position, skip) do
    {bytes, position, skip, _} = hash_round(bytes, instructions, position, skip)
    rounds(bytes, instructions, rounds - 1, position, skip)
  end

  def hash_round(bytes, instructions, position, skip, length \\ 256) do
    Part1.hash(bytes, length, instructions, position, skip)
  end

  def sparse_to_dense(bytes) do
    bytes
    |> Enum.chunk_every(16)
    |> Enum.map(&xor_chunk/1)
    |> :binary.list_to_bin
  end

  def xor_chunk(chunk) do
    Enum.reduce(chunk, &(&1 ^^^ &2))
  end

  def dense_to_hex(dense) do
    Base.encode16(dense, case: :lower)
  end
end
{% endhighlight %}

# Final thoughts
This was a fun albeit frustrating problem. The intermediate steps of part 2 were
difficult to debug and so it took quite a bit of time for silly reasons. It
turned out that rotating the list in part 1 made me confused in part 2. I am
tempted to rewrite at least parts of it.
