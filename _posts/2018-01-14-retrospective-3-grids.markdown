---
layout: post
title:  "Retrospective 3: Functional grids"
date:   2018-01-14 11:00:00 +0100
categories: retrospective
---

Quite a few of the problems require working with grids. As mentioned earlier, we
generally have no constant-time arrays in functional languages so the solution one
would write in imperative languages is not available here.

I have worked with grids in three different ways, which all have their pros and cons.

## First way: one-dimensional list
In my code for [Day 3](/aoc/{% post_url 2017-12-03-day3-spiral-memory %}) I chose to simply
not work with the grid in a 2D sense but instead unwrap the spiral and use one-dimensional offsets.

This is possible when there is a clear, ordered way to traverse the grid. The downside is that it requires
being able to convert two-dimensional deltas into one-dimensional ones. Because the spiral is of a known
width at each iteration this was possible.

In the end however I am not sure if the code was made simpler by doing this. When writing the solution I 
had not yet worked with grids in a functional way and so this was the most straight-forward way. The other 
possible solution was to work with a two-dimensional linked list but that would quickly get out of hand. I think
that a rewrite with the third way mentioned in this post would simplify the code greatly.

## Second way: two-dimensional linked list
When doing [Day 14](/aoc/{% post_url 2017-12-14-day14-disk-defragmentation %}) this was my first approach.
Essentially it involved using linked lists like you would arrays in an imperative language. Because access
is `O(n)` this will not be as efficient and the language rightly makes this slightly difficult.

When you only care about adjacent elements this is somewhat made easier by pattern matching as I tried to
use to my advantage on Day 14 but it was still way to complicated.

I later rewrote Day 14 with the third way of doing grids and it made both the code and the running time
very significantly better (a post on that might appear later).

## Third way: coordinate tuples in a map
The third way of doing grids, which I found when I actually researched the issue, is to store tuples of
coordinates in a map. This is especially suitable when:

* The grid is of unknown size or needs to expand. Then regular arrays wouldn't really be great even if they
would exist.
* The operations require non-sequential traversal. Using two-dimensional linked-lists works well enough
if you only care about adjacent cells in the same rows but breaks down as soon as you care about different rows
or cells which are not even adjacent. Coordinate tuples work great here.
* The grid consists of some empty and some filled cells. The map works great here and gives an efficient, sparse
representation.

The map is internally (usually) stored as a tree which means that lookup is `O(log n)`. This was quick enough
for all cases I used it on but gives an extra log factor compared to two-dimensional linked lists for some use cases.

## Overall: coordinate tuples are likely best
Everywhere that I used grids I think the third way would be the best. As mentioned I rewrote day 14 with that way
of doing grids and the flood fill code was significantly easier to write, easier to read and faster to run. Unless
there is a very good case to be made for doing anything else it is likely the first tool I will be reaching for when
doing similar things in the future.
