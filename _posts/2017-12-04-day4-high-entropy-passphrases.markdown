---
layout: post
title:  "Day 4: High Entropy Passphrases"
date:   2017-12-04 12:00:00 +0100
categories:
tags: easy
---
# Part 1

The first puzzle for today asks us to validate passwords. A valid password
consists of space-separated groups that are all unique.

(The full description can be found at
[http://adventofcode.com/2017/day/4](http://adventofcode.com/2017/day/4))

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day4.Part1 do
  def num_valid(passwords_string) do
    passwords_string
    |> String.trim
    |> String.split("\n")
    |> Enum.count(&valid?/1)
  end

  def valid?(password) do
    split = String.split(password, " ")

    Enum.count(split) == Enum.count(Enum.uniq(split))
  end
end
{% endhighlight %}

Essentially we just split the string at each space and use the `Enum` module to
count the number of elements as well as the unique elements (if a list has only
unique elements these two counts should be equal).

## Part 2
The second part makes the restrictions a bit more strict. Instead of groups
simply being unique, we now also require there to not be any groups that are
anagrams of each other.

My approach was to simply sort the characters in every group. Because anagrams
contain the exact same characters sorting the characters in two anagrams will
make them equal. Then I just did the same thing as in part 1.

Code:
{% highlight elixir %}
defmodule Aoc.Day4.Part2 do
  def num_valid(passwords_string) do
    passwords_string
    |> String.trim
    |> String.split("\n")
    |> Enum.count(&valid?/1)
  end

  def valid?(password) do
    split = String.split(password, " ")
    sorted = split
      |> Enum.map(&(String.split(&1, "")))
      |> Enum.map(&Enum.sort/1)
      |> Enum.map(&Enum.join/1)

    Enum.count(sorted) == Enum.count(Enum.uniq(sorted))
  end
end
{% endhighlight %}


# Final thoughts
This was not a very difficult problem to me. The solution is not as short and
concise as it could be but it is still fairly decent. I might go back later and
deduplicate the two parts a bit, though.
