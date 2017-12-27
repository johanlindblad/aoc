---
layout: post
title:  "Day 23: Coprocessor Conflagration"
date:   2017-12-23 12:00:00 +0100
categories:
tags: medium
---
# Part 1
Day 23 revisits the virtual machine from day 18 with some slight adjustments.
The question is how many times the `mul` instructions is executed.

My code is mostly a rewritten day 18 with a `num_muls/1` method to filter over
the simulation stream and count the number of `mul` instructions:

{% highlight elixir %}{% raw %}
defmodule Aoc.Day23.Part1 do
  require Record

  def preprocess(input), do: input |> String.trim |> String.split("\n") |> Enum.map(&preprocess_row/1)
  def preprocess_row(row), do: row |> String.trim |> String.split(" ")

  @initial_registers Enum.map(?a..?h, fn(_) -> 0 end)
  Record.defrecord :machine, state: :running, registers: @initial_registers, send_queue: [], receive_queue: [], program: [], program_past: []

  def num_muls(simulation_stream) do
    simulation_stream
    |> Stream.filter(fn(machine(program: [[ins | _] | _],)) ->
      ins == "mul"
    end)
    |> Enum.count
  end

  def simulation_stream(instructions) do
    Stream.iterate(machine(program: instructions), &step/1)
    |> Stream.take_while(&(machine(&1, :state) == :running))
  end

  def step(machine = machine(state: :halted)), do: machine
  def step(machine = machine(program: [])), do: machine(machine, state: :halted)
  def step(machine = machine(program: [instruction | rest], state: :running, program_past: past)) do
    machine(machine, program_past: [instruction | past], program: rest)
    |> step(instruction)
    |> maybe_halted
  end
  def step(machine = machine(program: [])), do: machine(machine, state: :halted)

  def step(machine = machine(registers: registers), ["set", a, b]) do
    registers = List.replace_at(registers, register_index(a), value(machine, b))
    machine(machine, registers: registers)
  end

  def step(machine = machine(registers: registers), ["sub", a, b]) do
    registers = List.update_at(registers, register_index(a), &(&1 - value(machine, b)))
    machine(machine, registers: registers)
  end

  def step(machine = machine(registers: registers), ["mul", a, b]) do
    registers = List.update_at(registers, register_index(a), &(&1 * value(machine, b)))
    machine(machine, registers: registers)
  end

  def step(machine = machine(program: program, program_past: past), ["jnz", a, b]) do
    case {value(machine, a), value(machine, b)} do
      {iff, _} when iff == 0 -> machine
      {_, jump} when jump < 0 ->
        {moved, rest} = Enum.split(past, 1 - jump)
        machine(machine, program: Enum.reverse(moved) ++ program, program_past: rest)
      {_, jump} when jump > 0 ->
        {moved, rest} = Enum.split(program, jump - 1)
        machine(machine, program: rest, program_past: Enum.reverse(moved) ++ past)
    end
  end

  def maybe_halted(machine = machine(program: [])), do: machine(machine, state: :halted)
  def maybe_halted(machine = machine()), do: machine

  def value(machine(registers: registers), spec = <<char>>) when char in ?a..?z, do: Enum.at(registers, register_index(spec))
  def value(_machine, spec), do: spec |> String.to_integer

  def register_index(<<register_name>>) when register_name in ?a..?z, do: register_name - ?a
end
{% endraw %}{% endhighlight %}

# Part 2
This part 2 is interesting in that it is not really about writing code but
rather about understanding the code we are given (as the puzzle input). When
setting register `a` to `1` as instructed, the code that executed very quickly
in part 1 now seemingly runs forever. We need to figure out what it is doing in
order to come up with the answer it would have found if given sufficient time.

Here is a first pass through my puzzle input, trying to structure it into
sections based on jumps that are performed:

{% highlight nasm %}{% raw %}
init:
set b 93
set c b
jnz a 2
jnz 1 5
mul b 100
sub b -100000
set c b
sub c -17000

main:
set f 1
set d 2

    inner:
    set e 2

        inner2:
        set g d
        mul g e
        sub g b
        jnz g 2
        set f 0
        sub e -1
        set g e
        sub g b
        jnz g -8 # jmp inner2

    sub d -1
    set g d
    sub g b
    jnz g -13 # jmp inner

jnz f 2
sub h -1
set g b
sub g c
jnz g 2
jnz 1 3
sub b -17
jnz 1 -23
{% endraw %}{% endhighlight %}

This does not make that much more sense but it is a good base to build from. The
next step is to find some common patterns and simplify them:

* We see that register `g` is only used when jumps are to be performed and it
is seemingly then used to subtract one value from another, before comparing the
result to `0`. This means that something like `a == b` would be performed as
`set g a`, `sub g b`, `jnz g 1`. If we had a `jeq` instructions (as is common in 
real architectures) that compare two values to each other instead of one value
to zero, this sequence could be rewritten as `jeq a b 1`. Furthermore we can
pretend we have `jne`, which is the opposite (jump if not equal). 
* `jnz` is sometimes used with a constant first operand in order to force a
jump. If we had a `jmp` instruction that simply jumped that amount of
instructions we could simplify things like `jnz 1 -23` into `jmp -23`.
* `sub` is sometimes used with negative numbers in order to add, which increases
the cognitive load while trying to understand the code. We can pretend we have
`add` has well.

If we use these simplifications and also allow ourselves some inline math and
jumping to labels we get something like the following, which starts do make
sense:

{% highlight nasm %}{% raw %}
init:
set b (93*100)+10000
set c b+17000

main:
set f 1
set d 2

    inner:
    set e 2

        inner2:
        jeq (d*e) b 2
            set f 0
        add e 1
        jne e b inner2

    add d 1
    jeq d b inner

jnz f 2
    add h 1

jne b c 2
    jmp halt
add b 17
jmp main

halt:
{% endraw %}{% endhighlight %}

Now we start to see the pattern a bit more clearly:
* The main loop executes with b from (9300+10000) until it equals the value of c
(9300+10000+17000).
* On each iteration of `inner` we start with `d=2` and `e=2`. We increase `e` until it
reaches the value of `b` and then rerun the loop from `d=3` and `e=2`, until `d`
reaches `b`.
* In the inner inner loop we check if `d*e == b` and if so, set `f` to 0. Later
on, we increase `h` by one if `f == 0`.

As we can now see, this is a prime number check. For each number from `b` up to
`c`, we check if it is a composite number and if it is, we set `f` to `0` (and
so we increase `h` by one).

The program is then asking: how many numbers `n` (`b <= n <= c`) are composite
numbers (not prime).

I implemented this by running the machine for the first few instructions in
order to setup `b` and `c`, and then perform the composite number check:

{% highlight elixir %}{% raw %}
defmodule Aoc.Day23.Part2 do
  alias Aoc.Day23.Part1
  require Aoc.Day23.Part1

  def num_primes(instructions) do
    s = "set a 1\n" <> instructions |> Part1.preprocess |> Part1.simulation_stream
    machine = s |> Enum.at(8)
    [_, b, c | _] = Part1.machine(machine, :registers)

    Stream.iterate(b, &(&1 + 17))
    |> Stream.take_while(&(&1 <= c))
    |> Stream.filter(fn(b) ->
      up_to = :math.sqrt(b) |> Float.ceil |> round
      Enum.any?(2..(up_to), &(rem(b, &1) == 0))
    end)
    |> Enum.count
  end
end
{% endraw %}{% endhighlight %}

# Final thoughts
This was a fun, and different, puzzle. Having to figure out what the assembly
code was doing was really interesting and the positive feeling of finally
"getting it" surpassed that of earlier days.

I would like to come back to this puzzle and implement it as a [peephole
optimization](https://en.wikipedia.org/wiki/Peephole_optimization) but other
than that I am quite happy with the code
