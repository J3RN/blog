---
layout: post
title: "Profiling Elixir for Advent of Code"
date: 2021-12-05T15:34:07-05:00
description: |
  Every now and again, you may run into a problem that isn't hard to solve using functional programming, but which is hard to solve _quickly_ using functional programming. For me, today, that problem was Advent of Code, Day 5.
tags:
- elixir
- profiling
- performance
---

Every now and again, you may run into a problem that isn't hard to solve using functional programming, but which is hard to solve _quickly_ using functional programming. For me, today, that problem was [Advent of Code], [Day 5].

If you're participating in Advent of Code and you haven't yet solved Day 5, this is your last chance to leave the page without spoilers. You have been warned!

## The Problem

You are provided with a list of lines of hydrothermal vents on the ocean floor. These lines are represented by start and end coordinates, such as `0,9 -> 5,9`. In this example, this means that there are hydrothermal vents at coordinates `0,9`, `1,9`, `2,9`, `3,9`, `4,9`, and `5,9`. Interestingly, two "lines" of vents may intersect or overlap, which create areas of increased concern.

For instance, given the input

```
0,9 -> 5,9
8,0 -> 0,8
9,4 -> 3,4
2,2 -> 2,1
7,0 -> 7,4
6,4 -> 2,0
0,9 -> 2,9
3,4 -> 1,4
0,0 -> 8,8
5,5 -> 8,2
```

You could visualize the map of the ocean floor as such:

```
1.1....11.
.111...2..
..2.1.111.
...1.2.2..
.112313211
...1.2....
..1...1...
.1.....1..
1.......1.
222111....
```

Here, each `.` represents a coordinate where there are no vents, each `1` represents that there is one line of vents that occupies that coordinate, `2` represents that there are two lines of vents that occupy that coordinate, etc.

The goal is to calculate the overall number of coordinates of increased concern (i.e. spaces whose number is `2` or higher).

## My First Solution

For starters, I parsed the input into a list of tuples of tuples, something like this:

```elixir
coordinate_pairs =
  [
    {{0, 9}, {5, 9}},
    {{8, 0}, {0, 8}},
    # ...
```

Next, I generated a list of lists to represent the ocean floor. I found the maximum x coordinate and maximum y coordinate in the input data, and generated the list of lists like so:

```elixir
ocean_floor_map = List.duplicate(List.duplicate(0, max_x + 1), max_y + 1)
```

Easy. Next, my idea was to perform a `reduce` where each pair of coordinates would be "applied" to the `ocean_floor_map` (details on that to come). Something like this:

```elixir
finished_ocean_map = Enum.reduce(pairs, ocean_floor_map, &apply_pair/2)
```

OK, simple. For the implementation of `apply_pair/2`, I used Elixir's [`for` macro]—it's list comprehension construct—to generate all coordinate pairs between the start coordinate and the end coordinate, and then used [`update_in/3`] to increment the value at those locations. It looked like this:

```elixir
def apply_pair(coordinate_pair, ocean_floor_map) do
  case coordinate_pair do
    {{x, y1}, {x, y2}} ->
      Enum.reduce(y1..y2, ocean_floor_map, fn y, acc ->
        update_in(acc, [Access.at(x), Access.at(y)], &(&1 + 1))
      end)

    {{x1, y}, {x2, y}} ->
      Enum.reduce(x1..x2, ocean_floor_map, fn x, acc ->
        update_in(acc, [Access.at(x), Access.at(y)], &(&1 + 1))
      end)

    {{x1, y1}, {x2, y2}} ->
      Enum.reduce(Enum.zip(x1..x2, y1..y2), ocean_floor_map, fn {x, y}, acc ->
        update_in(acc, [Access.at(x), Access.at(y)], &(&1 + 1))
      end)
  end
end
```

Once we have our `finished_ocean_map`, calculating the number or coordinates of concern was pretty simple:

```elixir
Enum.count(List.flatten(finished_ocean_map), &(&1 > 1))
```

_Voila_. Only one small problem: This solution is _slow_. For part one, the code only needed to consider horizontal or vertical lines of vents (i.e. not diagonal vent lines). On my laptop, this solution took nearly three seconds to run. Pretty slow. For part two, we consider all this lines. On my laptop, this solution took around 5.5 seconds—long enough for me to sit around twiddling my thumbs impatiently.

## Improving Performance

In order to figure out why my solution was so godawful slow, I used a built-in Elixir profiling tool, [`mix profile.eprof`], to profile part one. There are several built-in profiling tools, but I chose `eprof` as it gives me both time consumed (in some unit and also as a fraction of the whole) and also number of calls. The output looked like this:

```
Profile results of #PID<0.112.0>
#                                                   CALLS     %     TIME µS/CALL
Total                                           108705146 100.0 12921412    0.12
:gen_server.call/3                                      1  0.00        0    0.00
:erlang.list_to_tuple/1                                 1  0.00        0    0.00
:erlang.dt_spread_tag/1                                 1  0.00        0    0.00
...
:lists.do_flatten/2                                983071  1.67   215602    0.22
:lists.reverse/2                                   207142  1.96   253151    1.22
Access.get_and_update_at/5                      102804415 90.37 11677306    0.11

Profile done over 82 matching functions
```

The output is truncated because it is _long_. Weirdly, in my opinion, `eprof` puts the function that consumes the most time at the bottom. In any case, that function turned out to be `Access.get_and_update_at/5`, using 90.37% of the total running time! Wow!

My old friend `Access` had betrayed me. The solution was too elegant, I thought, and I needed to resort to more limited `List` module functions. I changed the lines
```elixir
update_in(acc, [Access.at(y), Access.at(x)], &(&1 + 1))
```

to

```elixir
List.update_at(acc, y, fn row -> List.update_at(row, x, &(&1 + 1)) end)
```

This does functionally the same thing, but is a bit more verbose. Let's profile it!

```
Profile results of #PID<0.112.0>
#                                                   CALLS     %     TIME µS/CALL
Total                                           107395276 100.0 21435654    0.20
:gen_server.call/3                                      1  0.00        0    0.00
:erlang.list_to_tuple/1                                 1  0.00        0    0.00
:erlang.dt_spread_tag/1                                 1  0.00        0    0.00
...
Enum."-count/2-lists^foldl/2-0-"/3                 981091  0.93   198430    0.20
:lists.do_flatten/2                                983071  0.95   203816    0.21
List.do_update_at/3                             102804415 96.23 20627578    0.20

Profile done over 78 matching functions
```

This solution is managed to shave a couple hundred milliseconds off the runtime, but wasn't the substantial gain I was looking for. Part two still took almost five seconds to run!

It was then that I realized that I had completely missed the point. In languages like C with "true" arrays and mutation, a solution resembling this one would be blazing fast. After all, finding the point we need to update in our matrix would be some trivial pointer arithmetic, followed by around three instructions to actually perform the update. Quick and easy. However, in Elixir, as with many other functional languages, we are working with _immutable datastructures_—meaning that we can't _update_ them, we can only make a copy resembling the first but with some differences.

In this case, if I was performing an operation such as `update_in(acc, [Access.at(9), Access.at(5)], &(&1 + 1))`, the runtime needs to create a new list for row 9, copying all values for each column except the value for column 5, which needs to be incremented. Since a row in the ocean floor map has been "modified," the runtime also needs to create a whole new ocean floor map. If the runtime is clever, the other rows can be reused since they haven't changed. Either way, this is obviously a lot more work than the efficient C-style solution.

But what if we changed our `ocean_floor_map` datastructure to something that handles updates more efficiently than a list of lists? Let's try a `Map`, 
where the keys represent coordinates and the values are the number of vent lines intersection that spot (initialized to zero):

```elixir
ocean_floor_map = for x <- 0..(max_x + 1), y <- (0..max_y + 1), into: %{}, do: {{x, y}, 0}
```

Now that we're using a `Map` for our `ocean_floor_map`, we'll need to update our `apply_pair` function accordingly. For this, I changed instances of

```elixir
List.update_at(acc, y, fn row -> List.update_at(row, x, &(&1 + 1)) end)
```

to

```elixir
Map.update!(acc, {x, y}, &(&1 + 1))
```

That's actually pretty clean! Last bit, let's update our counting code:

```elixir
Enum.count(finished_ocean_map, fn {_coordinate, vent_count} -> vent_count > 1 end)
```

That's pretty clean too! We were able to get rid of our `List.flatten` call.

Alright, time to profile it:

```
Profile results of #PID<0.112.0>
#                                                 CALLS     %   TIME   µS/CALL
Total                                           7343728 100.0 224661      0.31
:gen_server.call/3                                    1  0.00      0      0.00
:erlang.list_to_tuple/1                               1  0.00      0      0.00
:erlang.dt_spread_tag/1                               1  0.00      0      0.00
...
anonymous fn/1 in Day5.apply_pair/2               48900 11.25 252832      5.17
:maps.fold_1/3                                   983073 13.20 296612      0.30
anonymous fn/3 in Day5.problem1/0                983072 14.51 325883      0.33

Profile done over 79 matching functions
```

You may have noticed that updating the ocean floor map is no longer dominating our running time. Also, in practice this dropped the running time for part one and part two to 1.5 seconds 1.8 seconds, respectively. Not bad!

I knew we could do even better, though. When I was writing this solution, I remembered that [`Map.update!/3`] has a companion form, [`Map.update/4`]. The primary difference between these two is their behavior when the specified key does not exist in the map. In the former, a runtime error is raised. In the latter, the "default" argument will populated into the map as the value for the key instead of running the function argument. By switching from `Map.update!/3` to `Map.update/4`, we could avoid storing data for coordinates where no vents exist! The less data we store, the faster ~~updating~~ copying will be.

First, let's simplify our `ocean_floor_map` initialization:
```elixir
ocean_floor_map = %{}
```

Love it! 💖

Alright, and let's update our `Map.update!/3` to a `Map.update/4`:
```elixir
Map.update(acc, {x, y}, 1, &(&1 + 1))
```

Simple enough! And, finally, let's profile it:

```
Profile results of #PID<0.112.0>
#                                                CALLS     %  TIME µS/CALL
Total                                           821805 100.0 24199    0.29
:gen_server.call/3                                   1  0.00     0    0.00
:erlang.list_to_tuple/1                              1  0.00     0    0.00
:erlang.dt_spread_tag/1                              1  0.00     0    0.00
...
anonymous fn/3 in Enum.count/2                   97075  9.76 23614    0.24
:maps.fold_1/3                                   97076 14.82 35873    0.37
Map.update/4                                    103571 31.17 75427    0.73

Profile done over 58 matching functions
```

This resulted in part one and part two completing in 0.63 seconds and 0.75 seconds, respectively. Seeing as the BEAM takes roughly 500 ms to launch on my laptop anyhow, I'm very satisfied with this performance.

You can find [the finished code for the solution on GitHub](https://github.com/j3rn/advent-of-code/blob/main/2021/day5/day5.ex).

I hope you enjoyed taking this adventure with me, and happy hacking!

[Advent of Code]: https://adventofcode.com/2021/
[Day 5]: https://adventofcode.com/2021/day/5
[`mix profile.eprof`]: https://hexdocs.pm/mix/Mix.Tasks.Profile.Eprof.html
[`for` macro]: https://hexdocs.pm/elixir/Kernel.SpecialForms.html#for/1
[`update_in/3`]: https://hexdocs.pm/elixir/Kernel.html#update_in/3
[`Map.update!/3`]: https://hexdocs.pm/elixir/Map.html#update!/3
[`Map.update/4`]: https://hexdocs.pm/elixir/Map.html#update/4
