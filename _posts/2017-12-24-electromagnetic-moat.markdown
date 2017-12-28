---
layout: post
title:  "Day 24: Electromagnetic Moat"
date:   2017-12-24 12:00:00 +0100
categories:
tags: medium
---
# Part 1
Today we are tasked with building a bridge of what sounds like dominoes. We are
looking for the strongest bridge, with strength being the total number of
"spots". We are told we are starting with a `0`.

My solution simply keeps a collection of candidates and then in each iteration
generates new candidates by using all the different pieces that can be added to
the end.

The state that is kept for each candidate is a tuple of `{remaining pieces,
current score, outward-facing spots}`. This is enough to be able to know which
pieces have not yet been used and which number of spots we require to "match"
with the bridge in its current state.

We then keep track of these candidates and the current max score so as to be
able to generate new candidates until we reach the end and then know what the
maximum score was.

The code is as follows:

{% highlight elixir %}{% raw %}
defmodule Aoc.Day24.Part1 do
  def parse(input) do
    pairs = input |> String.trim |> String.split("\n") |> Enum.map(&parse_row/1) |> Enum.into(MapSet.new())

    map = Enum.reduce(pairs, %{}, fn({a,b}, map) ->
      Map.update(map, a, MapSet.new([{a,b}]), fn(set) -> MapSet.put(set, {a,b}) end)
      |> Map.update(b, MapSet.new([{a,b}]), fn(set) -> MapSet.put(set, {a,b}) end)
    end)

    {map, {pairs, 0, 0}}
  end

  def parse_row(row) do
    row |> String.split("/") |> Enum.map(&String.to_integer/1) |> List.to_tuple
  end

  def solve({map, initial_state}), do: max_score([initial_state], map)

  # state = {remaining, score, port}
  def max_score(state_list, map, max \\ 0)
  def max_score([], _, max), do: max
  def max_score(state_list, map, max) do
    {next_states, max} = Enum.reduce(state_list, {[], max}, fn({remaining, score, port}, {next_states, max}) ->
      Enum.filter(_options = map[port], fn(other) ->
        MapSet.member?(remaining, other)
      end)
      |> Enum.reduce({next_states, max}, fn({a,b}, {next_states, max}) ->
        contact_score = a + b
        remaining = MapSet.delete(remaining, {a,b})
        new_state = {remaining, score + contact_score, contact_score - port}
        max = max(max, score + contact_score)
        {[new_state | next_states], max}
      end)
    end)

    max_score(next_states, map, max)
  end
end
{% endraw %}{% endhighlight %}

# Part 2
In part 2 we do not want the max score of all bridges, but simply the max score
of the longest ones. Because the candidate in each iteration all have the same
length we can simply iterate until we get no new candidates and then check what
the maximum score was among the candidates of the last round.
{% highlight elixir %}{% raw %}
defmodule Aoc.Day24.Part2 do
  def solve({map, initial_state}), do: max_score([initial_state], map)

  # state = {remaining, score, port}
  def max_score(state_list, map) do
    next_states = Enum.reduce(state_list, [], fn({remaining, score, port}, next_states) ->
      extra = Enum.filter(map[port], fn(other) ->
        MapSet.member?(remaining, other)
      end)
      |> Enum.map(fn({a,b}) ->
        contact_score = a + b
        remaining = MapSet.delete(remaining, {a,b})
        {remaining, score + contact_score, contact_score - port}
      end)

      extra ++ next_states
    end)

    case next_states do
      [] -> Enum.max_by(state_list, fn({_,score,_}) -> score end) |> elem(1)
      _ -> max_score(next_states, map)
    end
  end
end
{% endraw %}{% endhighlight %}

# Final thoughts
This was a fun puzzle. At first I thought I was going to be able to use [dynamic
programming](https://en.wikipedia.org/wiki/Dynamic_programming) to solve this
efficiently as this problem seemed to have the "flavor" required. However, it
turns out that because subproblems are not independent (a subset of dominoes
might need a domino from a different subset) that is seemingly not possible.

There are a few other things I would like to explore however. Firstly, in my
puzzle input some dominoes have the form `x/x`. To keep the [branching
factor](https://en.wikipedia.org/wiki/Branching_factor) down these could use
some special handling; if we are at a `0/x` junction and we have the pieces
`x/x` and `x/y` there is no use generating the `0/x x/y` bridge because it will
always be worse than `0/x x/x x/y`. Even worse, both these bridges will end up
generating the same candidate bridges in the future (duplicating the work
required). If there are `n` such duplicates, we are possibly generating `2^n` as
many bridges as we need to.

My current thought is to eagerly use all `x/x` pieces whenever possible. Other
possibilities might be to remove all such duplicates from the set of dominoes
and adjust the scoring of all bridges which contain `_/x` junctions.

One more thing to keep the branching factor down might be to filter the set of
candidates to keep them unique. The only factors determining future
possibilities of a bridge are:
1. The set of dominoes left (or equivalently, the set of dominoes used)
2. The number of spots on the outwardly facing part of the last piece

Looking at my puzzle input there are likely many candidate bridges which are
equivalent in these regards (but just contain different orderings). In the same
way as the duplicates give rise to `2^n` possibilities, these different
permutations of the same pieces lead to the same issue.

Overall, I am somewhat happy with the solution. It should be cleaned up a bit
though, to make it easier to read (it was written in a rush on Christmas morning).
