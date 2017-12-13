---
layout: post
title:  "Day 1: Inverse Captcha"
date:   2017-12-01 12:00:00 +0100
categories: easy
---

## Part 1

For the first part today we are tasked with solving a captcha that essentially
has us sum numbers which are equal to their neighbours. The only slight
complication is that the list wraps around, so the last number should be
compared to the first.

(The full description can be found at
[http://adventofcode.com/2017/day/1](http://adventofcode.com/2017/day/1))

My solution is as follows:

{% highlight elixir %}
defmodule Aoc.Day1.Part1 do
  def captcha(digit_string) do
    digits = String.trim(digit_string) |> String.graphemes |> Enum.map(&String.to_integer/1)
    inner_captcha(digits)
  end

  def inner_captcha(digits), do: inner_captcha(digits, [])
  def inner_captcha([first, second | rest], remains) when first == second do
    first + inner_captcha([second | rest], remains ++ [first])
  end
  def inner_captcha([first, second | rest], remains), do: inner_captcha([second | rest], remains ++ [first])
  def inner_captcha([first], [other_first | _remains]) when first == other_first, do: first
  def inner_captcha([_first], [_other_first | _remains]), do: 0
end
{% endhighlight %}

The `captcha` function simply trims and splits the string as well as converting
each character to an integer. Then it calls the `inner_captcha` function, which
when looking at the code a few days later strikes me as needlessly complicated.

Anyway, the only thing of note is that the second parameter is a list of
processed elements. This is how the captcha can be calculated for the last
character.

# Part 2

Now we are looking ahead one-half of the length of the list instead of one
character. The full code is as follows:

{% highlight elixir %}
defmodule Aoc.Day1.Part2 do
  def captcha(digit_string) do
    digits = String.trim(digit_string) |> String.graphemes |> Enum.map(&String.to_integer/1)
    half_length = Enum.count(digits) |> div(2)
    inner_captcha(digits, Enum.drop(digits, half_length))
  end 

  def inner_captcha([], _), do: 0
  def inner_captcha([first | rest], [other_first | other_rest]) when first == other_first do
    first + inner_captcha(rest, other_rest ++ [first])
  end
  def inner_captcha([first | rest], [_other_first | other_rest]), do: inner_captcha(rest, other_rest ++ [first])
end
{% endhighlight %}

This solution uses the same idea as the one for the first part, where we cycle
through the list and append elements to a secondary list. Now, the second list
is intialized by copying the first and dropping half of the elements and then we
drop the first element on each operation. This means that with the list
`[1,2,1,4]` the arguments for each operation will be:

1. `[1,2,1,4], [1,4]`
2. `[2,1,4], [4,1]`
3. `[1,4], [1,2]`
4. `[4], [2,1]`

# Final thoughts

Overall this is not the solution I am the most happy with but it was not a
particularly difficult problem so I will not spend too much time on it.
I will however likely merge these two solutions during one of my cleaning cycles
as the solution for part 2 can be used for part 1 just by dropping 1 element 
instead of half of the list.
