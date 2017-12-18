---
layout: post
title:  "Day 7: Recursive Circus"
date:   2017-12-07 12:00:00 +0100
categories:
tags: medium revisit
---
# Part 1
Today's problem essentially consists of a (non-binary) tree of "discs"; we are
interested in finding its root.

My solution is quite simple:

{% highlight elixir %}
defmodule Aoc.Day7.Part1 do
  def parse_and_find(input) do
    parse_structure(input)
    |> root_of
  end

  def root_of(nodes) do
    node_names = nodes |> Enum.map(&(elem(&1, 0))) |> MapSet.new
    child_lists = nodes |> Enum.map(&(elem(&1, 1))) |> List.flatten

    Enum.reduce(child_lists, node_names, fn(child, nodes) ->
      MapSet.delete(nodes, child)
    end)
    |> MapSet.to_list
    |> List.first
  end

  def parse_structure(lines) do
    lines
    |> String.trim
    |> String.split("\n")
    |> Enum.map(&(parse_line(&1)))
  end
 
  def parse_line([name, _weight]), do: {String.to_atom(name), []}
  def parse_line([name, _weight, "->" | children]), do: {String.to_atom(name), Enum.map(children, &clean/1)}
  def parse_line(line), do: parse_line(String.split(line, " "))
  defp clean(child_string), do: child_string |> String.trim_trailing(",") |> String.to_atom
end
{% endhighlight %}

Basically we know that all child nodes will exist on the right hand of one row
(the row of its parent) and the root node will not be. For this reason it is
sufficient to just keep an initial set of all nodes, reduce over the
right hand sides and see which one single node remains.

# Part 2
In part 2 we need to do some more advanced things to the tree (I should have
known and build it in part 1 already). Basically the tree is "balanced" (the
recursive weights are equal between all children) save for one exception, which
we should find. 

My solution became quite long and consists of a few parts:

{% highlight elixir %}
defmodule Aoc.Day7.Part2 do
  alias Aoc.Day7.Part1

  def parse_and_find(input) do
    {_, tree} = Part1.parse_structure(input) |> build_tree
    weights = parse_weights(input)

    mismatch(tree, weights)
  end

  def mismatch(tree, _) when map_size(tree) == 0, do: 0
  def mismatch(tree, weights) do
    subweights = Map.keys(tree)
                 |> Enum.map(&(weight(tree, weights, &1)))

    if Enum.uniq(subweights) |> Enum.count > 1 do
      {_odd_weight, odd_index} = unique_element(subweights)
      odd_node = Map.keys(tree) |> Enum.at(odd_index)
      diff = Enum.max(subweights) - Enum.min(subweights)

      case mismatch(tree[odd_node], weights) do
        0 -> weights[odd_node] - diff
        mismatch -> mismatch
      end
    else
      Map.values(tree)
      |> Enum.map(&(mismatch(&1, weights)))
      |> Enum.sum
    end
  end

  def unique_element([head | tail]), do: unique_element(tail, {head, 0}, 0)
  def unique_element([candidate | tail], {candidate, index}, at), do: unique_element(tail, {candidate, index}, at + 1)
  def unique_element([head], _candidate, at), do: {head, at + 1}
  def unique_element([head | tail], {candidate, index}, at), do: unique_element(tail, {candidate, index}, {head, at + 1}, at + 1)
  def unique_element([candidate | _tail], {candidate, _}, {other, index}, _at), do: {other, index}
  def unique_element([other | _tail], {candidate, index}, {other, _}, _at), do: {candidate, index}

  def weight(tree, weights, key) do
    subtree_weight = 
      Map.keys(tree[key])
      |> Enum.map(&(weight(tree[key], weights, &1)))
      |> Enum.sum

    weights[key] + subtree_weight
  end

  def build_tree(descriptions), do: build_tree(Enum.into(descriptions, %{}), %{})
  def build_tree(structure_map, tree), do: build_tree(structure_map, tree, Part1.root_of(structure_map))
  def build_tree(structure_map, tree, node_name) do
    {children, structure_map} = Map.pop(structure_map, node_name)

    {structure_map, tree} = Enum.reduce(children, {structure_map, tree}, fn(child, {structure_map, tree}) ->
      build_tree(structure_map, tree, child)
    end)

    subtree = Map.take(tree, children)
    tree = Map.put(tree, node_name, subtree)
           |> Map.drop(children)

    {structure_map, tree}
  end

  def parse_weights(lines) do
    lines
    |> String.trim
    |> String.replace(~r/\(|\)|->/, "")
    |> String.split("\n")
    |> Enum.map(&(parse_line(&1)))
    |> Enum.into(%{})
  end

  defp parse_line([name, weight | _]), do: {String.to_atom(name), String.to_integer(weight)}
  defp parse_line(line), do: parse_line(String.split(line, " "))
end
{% endhighlight %}

The main thing is the `mismatch/2` function. This is a recursive function which
traverses own and looks at each node as if it was the root of its own tree.

I first calculate the weight of each child and check if there is more than one
value. If there is more than one value we know that there is a mismach among
one of the descendents and so I call the `mismatch/2` function on the odd one.

There are two cases:
* There is no mismatch here, because it is the direct descendent which has the
incorrect weight. If so I calculate the difference and return this.
* There is a mismatch here, in which case we can recurse down this subtree.

The other possibility is that there is no mismatch in this subtree. We arrive at
this conclusion when we run `mismatch/2` on a subtree without descendents, in
which case the first clause matches and returns 0.

The rest of the code consists of:
* The `unique_element/1` function, which I wrote mainly as a fun
pattern-matching/recursion exercise. This can be done in much less complicated
ways.
* The `weight/3` function, which is just a recursive weight calculation function
* `build_tree/3`, which builds the tree from the description. I want to do redo
this in a cleaner way. I also don't think that I actually need to find out the
root first before building, but that I can instead build the tree in parts and
let it sort itself out.

# Final thoughts
This was a fun puzzle, but I am not really happy with the solution; I will
definitely revisit this later. I have a thought in my head for how I can combine
`mismatch/2` and `weight/3` into the same function and likely save some unnecessary
repetitions.
