---
layout: post
title:  "Retrospective 2: Streams are great"
date:   2018-01-14 10:30:00 +0100
categories: retrospective
---
Solving a problem with streams has been a common pattern for me this year. I have found it to
be a great way of making a recursive solution more easily debuggable. When solving a problem
recursively (as often happens when writing functional code) the only real low-overhead debug
options are either:

1. Writing to the console, which as always is of limited use. If the depth is more than a
few or the state is complex this quickly becomes useless.
2. If the depth is explicitly given as a parameter, run with a lower value. In many of
the AoC problems this is not the case.

I quickly found streams to be a great alternative to these. There are various uses for streams
in Elixir but essentially I used them as what other languages would possibly call generators, i.e.

* A state
* A function to generate the next state

As a (contrived) example, think of calculating the Fibonacci sequence. If you want the 1000th
number in the sequence but there is something wrong in your code, the more straight-forward
version is not really helpful.

{% highlight elixir %}{% raw %}
defmodule Fib do
  def fib(0), do: 1
  def fib(1), do: 1
  def fib(n), do: fib(n-1) + fib(n-2)
end
{% endraw %}{% endhighlight %}

The stream-based version meanwhile, let's you use the `Stream` and `Enum` modules
to inspect intermediate values. The stream code itself is as follows:

{% highlight elixir %}{% raw %}
defmodule FibStream do
  @initial {0,1}

  def fib(state \\ @initial)
  def fib({a,b}), do: {b, a+b}

  def fib_state_stream, do: Stream.iterate(@initial, &Fib.fib/1)
  def fib_stream, do: fib_state_stream() |> Stream.map(&(elem(&1, 1)))
end
{% endraw %}{% endhighlight %}

You can then look at the 1000th number by running `FibStream.fib_stream() |> Enum.at(999)`,
the 10 first values by running `FibStream.fib_stream() |> Enum.take(10)` or inspect the
generator state by running `FibStream.fib_state_stream() |> Enum.at(1)`.

Essentially this lets you pretend to run the code in a VM which you can freely poke around in.
Of course this is not very useful (and instead definitely overkill) but it was of great use in
some of the more complex problems like [Day 19](/aoc{% post_url 2017-12-19-day19-a-series-of-tubes %}).

Furthermore this allows you to write code to process the stream to refine the data and
use it for other means. As an example, the video for Day 19 was made by taking the stream that's
used for calculating the answer and map each iteration to the text-based map. This would not be
as easy to do with the more straight-forward, tail-recursive version. As such streams are a great
way to encourage experimentation.

In my quick and unscientific tests, stream-based versions were about twice as slow as regular,
tail-recursive ones. For these kinds of problems it was nothing to worry about.
