---
title: 'Advent of Q'
description: ''
author: Stephen Taylor
date: December 2021
keywords: aoc, vectordojo
---
# The birth of the cool

The annual [Advent of Code](https://adventofcode.com) has become a programmers’ Christmas tradition: a daily series of puzzles of varying difficulty to solve and discuss.

This year’s series has raised the excitement level in the dojo because the puzzles are particularly susceptible to vector solutions. 
Here is a round-up of the first week in the q vector dojo. 

We nod to code golfers who seek the shortest solutions, and to others pursuing the fastest. 
But in the dojo we are looking for the most ‘vector-ish’ solutions. 
That’s admittedly a bit vague, because it is at bottom an aesthetic criterion. 
It’s most commonly pronounced “cool”.

Feel free to chime in with comments and alternatives: what follows is unlikely to be the last word in cool. 


## Day 1: Sonar Sweep

Every puzzle has the same first step: ingesting the data.
We‘re proud of how easy it is to convert each day’s text file into a tractable q data structure.
```q
q)show d:"J"$read0 `:day1.txt
148 167 168 169 182 188 193 209 195 206 214 219 225 219 211 215 216 195 200 1..
```
How many of these depth measurements are larger than the previous measurement?

The [Each Prior](https://code.kx.com/q/ref/maps/#each-prior) iterator applied to the Greater Than operator `>` derives a function `>':` that tells us exactly that. 
We use it with a zero left argument for the first comparison.
```q
q)0>':d
11111111011110011010111111010111111101100011111001101110110011100110111101110..
```
We’re not interested in the first comparison, so we discard it and count the remaining hits.
```q
q)sum 1_ 0>':d
1400i
```
Because we are not interested in the first comparison, the 0 could be any integer.
Perhaps better to get rid of it entirely, and apply `>':` as a unary, using either bracket or prefix syntax.
```q
q)sum 1_ >':[d] / bracket
1400i
q)sum 1_ (>':)d / prefix
1400i
```
Without the 0 left argument to `>':`, q has its own rules for what to compare the first item of `d` to. 
We don’t care, but you can read about these [defaults](https://code.kx.com/q/ref/maps/#each-prior).

We see above that the derived function `>':` is variadic: we can apply it as either a unary or a binary. 
Applying it as a unary means we could instead use the [`prior`](https://code.kx.com/q/ref/prior) keyword.
```q
q)sum 1 _ (>) prior d
1400i
```
That is better q style, and perhaps the coolest way to write the solution. 

Note the parens in `(>)`, which give it noun syntax. 
That is, the parser reads it as the left argument of `prior` rather than trying to apply it. 
With this as a model, part 2 looks simple. 
We want the 3-point moving sums of `d`, of which we shall ignore the first two.
```q
q)a:()!"j"$() / answers
q)a[`$"1-1"]:sum 1 _ (>) prior d
q)a[`$"1-2"]:sum 1 _ (>) prior 2 _ 3 msum d
q)show a
1-1| 1400
1-2| 1429
```
Oleg Finkelshteyn has a simpler (and faster) solution that baffles me.
```q
q)sum(3_d)>-3_d
1429
```
Factoring out the repeated 3s, I prefer this as
```q
q)sum .[>] (1 neg\3)_\:d
1429
```
but still cannot see why comparing `d[i]` to `d[i-3]` gives the same result as comparing the moving sums. Help, anyone?


## Day 2: Dive!

Today‘s text file consists of course adjustments that affect horizontal position and depth.
```txt
forward 3
down 7
forward 7
down 4
down 9
..
```
Take the starting position and depth as `0 0`. 
```q
q)forward:1 0*; down:0 1*; up:0 -1*
q)show c:value each read0`:day2.txt
3 0
0 7
7 0
0 4
0 9
0 7
..
```
The final position and depth are simply the sum of `c` and the answer to part 1 is their product.
```q
q)show a[`$"2-1"]:prd sum c
1561344
```
Part 2 complicates the picture. The first column of `c` still describes forward movements. But we now need to calculate ‘aim’. Up and Down now adjust aim. Depth changes by the product of forward motion and aim.

A table can help us to think this through.
```q
q)crs:{select cmd:x,fwd,ud from flip`fwd`ud!flip value each x}read0`:day2.txt
q)update aim:sums ud from `crs
`crs
q)update down:fwd*aim from `crs
`crs
q)show crs
cmd         fwd ud aim down
---------------------------
"forward 3" 3   0  0   0
"down 7"    0   7  7   0
"forward 7" 7   0  7   49
"down 4"    0   4  11  0
"down 9"    0   9  20  0
"down 7"    0   7  27  0
"forward 5" 5   0  27  135
"forward 9" 9   0  27  243
"forward 3" 3   0  27  81
"forward 8" 8   0  27  216
"down 4"    0   4  31  0
"down 6"    0   6  37  0
"down 3"    0   3  40  0
"forward 7" 7   0  40  280
"forward 1" 1   0  40  40
"forward 4" 4   0  40  160
"down 1"    0   1  41  0
..
```
Now we have the changes in horizontal and vertical position ``crs[`fwd`down]`` and can simply sum for the final position.
```q
q)sum each crs[`fwd`down]
2033 909225
```
But the `down` column is no more than the product of the `fwd` column and the accumulated sums of the `ud` column. 
We can express the whole thing in terms of the `fwd` and `ud`vectors.
```q
q)c:value each read0`:day2.txt
q)fwd:c[;0]; ud:c[;1] / forward; up-down
q)show a[`$"2-2"]:prd sum each(fwd;fwd*sums ud)
1848454425
```
Golfers will prefer `sum@/:` to `sum each` but good q style is to use the keywords where they improve legibility. 



