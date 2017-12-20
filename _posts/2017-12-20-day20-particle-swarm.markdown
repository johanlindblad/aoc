---
layout: post
title:  "Day 20: Particle Swarm"
date:   2017-12-20 12:00:00 +0100
categories:
tags: medium
---
# Part 1
Today we are observing particles modeled in linear motion with constant
acceleration and (possibly) variable speed and position.

The first question is regarding which particle will be closest (manhattan
distance) to `0,0,0` in the long run. My first naive solution worked even though 
I now know that it does not always work in the general case.

My thought was that the lowest acceleration will win out, tie-broken by low
initial velocity and position. As such I just sorted the tuples by the
(manhattan) distance and took the first one.

I will revisit this and do the proper calculus solution (as I did in part 2)
later.

Here is the code:

{% highlight elixir %}
defmodule Aoc.Day20.Part1 do
  def parse(input), do: input |> String.trim |> String.split("\n") |> Enum.map(&parse_row/1)
  def parse_row(row) do
    row |> String.split([",", "=", "<", ">", " "]) |> Enum.map(&maybe_int/1) |> List.flatten
    |> Enum.chunk_every(3) |> Enum.map(&List.to_tuple/1)
  end

  def maybe_int(string) do
    case Integer.parse(string) do
      {n, _} -> [n]
      :error -> []
    end
  end

  # TODO: not correct in general case
  # use math similar to part 2
  def closest_long_term(particles) do
    {_particle, index} = particles |> Enum.with_index |> Enum.min_by(fn
      ({[p, v, a], _i}) -> {manhattan(a), manhattan(v), manhattan(p)}
    end)

    index
  end

  def manhattan({x, y, z}), do: abs(x) + abs(y) + abs(z)
end
{% endhighlight %}

# Part 2
In part two we are interested in knowing which particles will collide with each other
and which will remain (in this discrete-time simulation).

I chose to take the calculus route and actually calculate rather than simulating
each step (I half-expected the puzzle input to make this infeasible).
The formulas are slightly different in the discrete case which
caught me out at first but are essentially the following (I will skip the
intermediate steps):

```
a = a_0
v = v_0 + a_0*t
x = x_0 + v_0*t + a_0*t(t+1)/2
```

Where `a_0` is the intial acceleration (in our case constant), `v_0` the initial
velocity, `x` is the position (in this case in the x dimension) and t is time.

What differs from the continuous case is the term `a*t(t+1)/2`, which in the
continuous case would be `at^2`. This arrives from the fact that our velocities
are essentially staircase functions where we add `a_0` at each step, so that:

```
v(0) = v_0
v(1) = v_0 + a_0
v(2) = v_0 + a_0 + a_0
```

Summing these gives `v_0`, plus `a_0` times the arithmetic sum `1 + 2 + ... + t`
which equals `t(t+1)/2`.

Given these equations for the position we can subtract two points (to get their
distance) and solve the quadratic equation for 0. My code solves the quadratic
equation with the quadratic formula (with handling of a few special cases) and
then verifies all integer solutions. It then does this for all three dimensions
and checks if there is a `t` where the distance is 0 in all dimensions.

I then check all pairs of particles and exclude the ones that collide. Once
again, my solution works in this case but not in the general one; if a particle
is destroyed it can not destroy a different particle in the future. I should revisit this to group the particles by when they collide and handle all
conflicts in order.

Anyway, the code is as follows:

{% highlight elixir %}
defmodule Aoc.Day20.Part2 do
  # TODO: sort by time and avoid having dead particles collide in the future
  def particles_left(particles) do
    set = MapSet.new(particles)

    left = Enum.reduce(set, set, fn(particle, left) ->
      rest = MapSet.delete(set, particle)
      collides_with = Enum.filter(rest, &(collide_at(&1, particle))) |> MapSet.new()

      case MapSet.size(collides_with) do
        0 -> left
        _ -> MapSet.delete(left, particle)
      end
    end)

    MapSet.size(left)
  end

  def collide_at(p1, p2) do
    # Once for each direction (x, y, z)
    times = Enum.flat_map(0..2, fn(i) ->
      collision_points(extract_component(p1, i), extract_component(p2, i)) end)
    |> Enum.filter(fn(t) -> 
      position(p1, t) == position(p2, t)
    end)

    case times do
      [] -> nil
      _ -> times |> Enum.min
    end
  end

  def collision_points(p1 = [_x1, _v1, _a1], p2 = [_x2, _v2, _a2]) do
    collision_candidates(p1, p2)
    |> Enum.filter(&(&1 >= 0))
    |> Enum.filter(&(position(p1, &1) == position(p2, &1)))
  end

  def collision_candidates([x1, v1, a1], [x2, v2, a2]) do
    # Coefficients for the quadratic equation (for twice the distance)
    {a1, b1, c1} = {a1, (2*v1) + a1, 2 * x1}
    {a2, b2, c2} = {a2, (2*v2) + a2, 2 * x2}
    {a, b, c} = {a2-a1, b2-b1, c2-c1}

    b2_4ac = (b*b - 4*a*c)

    cond do
      # Same acceleration and speed, no solutions or always solved depending
      # on initial position
      a == 0 && b == 0 -> []
      # Same acceleration, solved by (p2-p1) + (p2-p1)t = 0
      a == 0 && rem(c, b) == 0 -> [div(-c, b)]
      a == 0 && rem(c, b) != 0 -> []

      # Quadratic formula indicates no solutions
      b2_4ac < 0 -> []

      # Quadratic formula indicates only one solution
      b2_4ac == 0 -> [div(-b, 2*a)]

      # Quadratic formula indicates two possible solutions
      b2_4ac > 0 ->
        root = :math.sqrt(b2_4ac) |> round
        first_root = div(-b + root, 2*a)
        second_root = div(-b - root, 2*a)
        [first_root, second_root]
    end
  end

  def position(particle = [p, _v, _a], t) when is_tuple(p) do
    Enum.map(0..2, &(extract_component(particle, &1)))
    |> Enum.map(&(position(&1, t)))
  end
  def position([p, v, a], t) do
    p + (v*t) + (a*div(t*t + t, 2))
  end

  defp extract_component(particle, index), do: Enum.map(particle, &elem(&1, index))
end
{% endhighlight %}

# Final thoughts
This was a fun problem which had me do some calculus on pen and paper. As noted
above I should revisit this problem to:
1. Do a calculus solution for part 1 to actually get the correct answer in the
general case
2. Properly handle collisions of the same particle in different time steps
