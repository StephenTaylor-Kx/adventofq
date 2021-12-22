---
title: 'Advent of Q'
description: ''
author: Stephen Taylor
date: December 2021
keywords: aoc, vectordojo
---
# The birth of the cool

The annual [Advent of Code competition](https://adventofcode.com) has become a programmers’ Christmas tradition: a daily series of puzzles of varying difficulty to solve and discuss.

This year’s series has raised the excitement level in the Vector Dōjō because the puzzles are particularly susceptible to vector solutions. 
Here is a round-up of the first week. 

We nod to code golfers who seek the shortest solutions, and to others pursuing the fastest. 
But in the dōjō we are looking for the most ‘vector-ish’ solutions. 
That’s admittedly a bit vague, because it is at bottom an aesthetic criterion. 
(Usually pronounced “cool”.)

As always in the Vector Dōjō, we are more interested in exploring the process of discovery than in the solutions themselves. 
Feel free to chime in with comments and alternatives: what follows is unlikely to be the last word in cool. 


## [Day 1: Sonar Sweep](https://adventofcode.com/2021/day/1)

Our first puzzle shows off how easily q converts a text file to tractable data in memory, and explores some subtleties of the Each Prior iterator, including the mysteries of why you sometimes need to parenthesise a derived function.

### Ingestion

Every puzzle starts by ingesting a text file.

```txt
199
200
208
210
200
207
240
269
260
263
```
[Tok](https://code.kx.com/q/ref/tok/) and [`read0`](https://code.kx.com/q/ref/read0/) eat that up.
```q
q)show d:"J"$read0 `:test1.txt
199 200 208 210 200 207 240 269 260 263
```

### Part 1

How many of these depth measurements are larger than the previous measurement?

The [Each Prior](https://code.kx.com/q/ref/maps/#each-prior) iterator applied to the Greater Than operator `>` derives a function Greater Than Each Prior that tells us exactly that. Notice that the iterator `':` is applied *postfix*; that is, its argument, the Greater Than operator, is written on its left:  
```q
>':
```
A function derived this way (with postfix syntax) has *infix* syntax. We apply it with a zero left argument for the first comparison.
```q
q)0>':d
1111011101b
```
We’re not interested in the first comparison, so we discard it and count the remaining hits.
```q
q)sum 1_ 0>':d
7
```
Because we are not interested in the first comparison, the 0 could be any integer.
Perhaps better to get rid of it entirely.

Functions derived from Each Prior are [variadic](https://code.kx.com/q/basics/glossary/#variadic), which is to say that for a binary `f` you can apply `f':` as either a binary (as above) or as a unary, using either bracket or prefix syntax.
```q
q)sum 1_ >':[d] / bracket
7i
q)sum 1_ (>':)d / prefix
7i
```
**Detail:** those parentheses. Any function can be applied with bracket notation. So `>':` can be applied that way as a unary like any other unary, i.e. as `>':[d]`. Why can’t we apply it prefix like other unaries? For example, as `>':d`? 

The answer is above: applying the iterator *postfix*, derives a function `>':` that has *infix* syntax, which the parser won’t apply prefix. However, with parentheses round it, it has *noun* syntax – like a list – and, like any list, *can* be applied prefix.

**Extra detail**: *Any function can be applied with bracket notation.* Iterators are functions too and the rule applies to them as well. Normal practice is to apply an iterator postfix, but Greater Than Each Prior can also be written `':[>]`. (Not recommended: the unusual syntax is likely to confuse a reader.) Written this way, it is still variadic, but no longer has infix syntax, and so *can* be applied prefix as a unary.
```q
q)':[>][0;d]  / applied as binary (bracket)
1111011101b
q)':[>]d      / applied as unary (prefix)
1111011101b
```

Applying it as a unary, without the 0 left argument, q has its own rules for what to compare the first item of `d` to. 
In this problem, we don’t care, but you can read about these [defaults](https://code.kx.com/q/ref/maps/#each-prior).

Because we are applying Greater Than Each Prior as a unary, we could instead use the [`prior`](https://code.kx.com/q/ref/prior) keyword.
```q
q)sum 1 _ (>) prior d
7i
```
Using keywords is better q style, and perhaps the coolest way to write the solution. 

Note the parens in `(>)`, which give it noun syntax. 
The parser reads it as the left argument of `prior` and does not try to apply it. 

### Part 2

With this as a model, part 2 looks simple. 
We want the 3-point moving sums of `d`, of which we ignore the first two.
```q
a:()!"j"$() / answers
d:"J"$read0`:day1.txt`
a[`$"1-1"]:sum 1 _ (>) prior d
a[`$"1-2"]:sum 1 _ (>) prior 2 _ 3 msum d
```
Oleg Finkelshteyn has a simpler (and faster) solution that baffles me.
```q
sum(3_d)>-3_d
```
Why does comparing `d[i]` to `d[i-3]` give the same result as comparing the moving sums?


## [Day 2: Dive!](https://adventofcode.com/2021/day/2)

Today’s problem solution uses projections to ingest the data, then a table to think through a solution to the second part. Finally we reduce the table solution to a simple vector expression.

### Ingestion

The text file consists of course adjustments that affect horizontal position and depth.
```txt
forward 5
down 5
forward 8
up 3
down 8
forward 2
```
Take the starting position and depth as `0 0`. 
```q
q)forward:1 0*; down:0 1*; up:0 -1*
q)show c:value each read0`:test2.txt
5 0
0 5
8 0
0 -3
0 8
2 0
```

### Part 1

The final position and depth are simply the sum of `c` and the answer to part 1 is their product.
```q
q)prd sum c
150
```

### Part 2

Part 2 complicates the picture. The first column of `c` still describes forward movements. But we now need to calculate ‘aim’. Up and Down now adjust aim, not depth. Depth changes by the product of forward motion and aim.

A table can help us to think this through.
```q
q)crs:{select cmd:x,fwd,ud from flip`fwd`ud!flip value each x}read0`:test2.txt
q)update aim:sums ud from `crs
`crs
q)update down:fwd*aim from `crs
`crs
q)show crs
cmd         fwd ud aim down
---------------------------
"forward 5" 5   0  0   0
"down 5"    0   5  5   0
"forward 8" 8   0  5   40
"up 3"      0   -3 2   0
"down 8"    0   8  10  0
"forward 2" 2   0  10  20
```
Now we have the changes in horizontal and vertical position ``crs[`fwd`down]`` and can simply sum for the final position.
```q
q)sum each crs[`fwd`down]
15 60
```
But the `down` column is no more than the product of the `fwd` column and the accumulated sums of the `ud` column. 
We can express the whole thing in terms of the `fwd` and `ud` vectors.
```q
q)`fwd:`ud set'flip c  / forward; up-down
q)prd sum each(fwd;fwd*sums ud)
900
```
The repetition of `fwd` catches the eye. 
Isn’t `(fwd;fwd*sums ud)` just `fwd` multiplied by 1 and by `sums ud`?
```q
q)prd sum fwd*1,'sums ud
900
```
Or expressed as a function directly on the columns of `c`
```q
q)prd sum {x*1,'sums y}. flip c
```
That reduces our complete solution to
```q
forward:1 0*; down:0 1*; up:0 -1*
c:value each read0`:day2.txt
a[`$"2-2"]:prd sum c
a[`$"2-2"]:prd sum {x*1,'sums y}. flip c
```
The table operations were a helpful way to visualise the elements of the problem. On study, they refactored to a much simpler vector solution. 


## [Day 3: Binary Diagnostic](https://adventofcode.com/2021/day/3)

In today’s problem we 

*    abstract from two algorithms by passing an operator as argument
*    meet the Zen monks using the Do iterators
*    use the Do iterator to repeatedly apply a filter

### Ingestion

Ingestion here is a treat. 
We rely on the atomic iteration implicit in the Equal operator. 
Comparing the file lines to `"1"` gets us a boolean matrix.
```q
q)show dg:"1"=read0`:test3.txt / diagnostic
00100b
11110b
10110b
10111b
10101b
01111b
00111b
11100b
10000b
11001b
00010b
01010b
```

### Part 1

Finding the most common bits is just comparing the the sum of `dg` to half its count.
```q
q)sum[dg]>=count[dg]%2  / gamma bits
10110b
```
And the least-common bits are… not them.
```q
q)sum[dg]<count[dg]%2  / epsilon bits
01001b
```
> *— How many Zen monks does it take to change a lightbulb?*<br>
>— *Two: one to change it, one not to change it.* 

So I think of this pattern <code>1 f\\</code> of the [Do](https://code.kx.coom/q/ref/acclmulators/#do) iterator as the Zen monks. 
```q
q)1 not\sum[dg]>=count[dg]%2  / epsilon and gamma bits
10110b
01001b
```
All that remains is to encode these two binary numbers as decimals and get their product. 
```q
q)prd 2 sv'1 not\sum[dg]>=count[dg]%2
198
```
The [`sv`](https://code.kx.com/q/ref/sv/) keyword is a bit of a ‘portmanteau’ function. The common theme is scalar (atom) from vector. In the domain of integers it interprets a vector as a number in the base of its left argument. 

## Part 2

To find the oxygen generator rating we need to filter the rows of `dg` by an aggregation (most-common bit) of its first column, and so on. 
We are going to have to specify the iteration. 

A succession of filters suggests an [Accumulator](https://code.kx.com/q/ref/accumulators/) iteration, where the result of one iteration is the argument of the next. 

We’ll use a binary filter function. One argument will be a bit vector. The other will be a comparison operator corresponding to most-common bit (`>=`) or least-common bit (`<`).
```q
fltr:{[op;bits]where bits=.[op]1 .5*(sum;count)@\:bits}
```
Using the [Apply](https://code.kx.com/q/ref/apply/) operator to apply the comparison operator between the items of a pair allows us to pass the comparison operator to `fltr` as an argument. 

To work across the columns of `dg` we’ll flip it so its columns become the items of a list.
We’ll use the Scan iterator to filter the rows of `flip dg` until the next iteration would leave no rows left. 

We start with all the rows: `til count dg`. We want some binary function `f` so our iteration is
```q
(til count dg)f/flip dg
```
The function `f` we want will take as its left argument a list of row indices. Its right argument will be a bit vector – the successive columns of `dg`. Its result will be a list of row indices to be used for the next iteration. We need it to stop filtering before the last indices are gone.
Here’s `f`:
```q
(til count dg){$[count i:fltr[>=]y x;x i;x]}/flip dg
```
Here we can see the structure of the iteration. 
The initial state `til count dg` is a list of all the rows of`dg`.
The lambda being iterated tests the first column of `dg` and returns the rows that pass the test. 
Only the rows listed in the left argument are tested, so eliminated rows stay eliminated. 

We can use the Scan form of the Do iterator to watch the rows being filtered. 
```q
q)dg(til count dg){$[count i:fltr[>=]y x;x i;x]}\flip dg
(11110b;10110b;10111b;10101b;11100b;10000b;11001b)
(10110b;10111b;10101b;10000b)
(10110b;10111b;10101b)
(10110b;10111b)
,10111b
```
That is the oxygen generator rating for the test data. 
Switching the comparison operator will give us the CO2 scrubber rating.
Let’s make the comparison operator the lambda’s third argument, and embed the iteration in a function that returns the winning row.
```q
q)analyze:{y first(til count y){$[count i:fltr[z;] y x;x i;x]}[;;x]/flip y}
q)analyze[>=] dg
10111b
```
Afterthoughts: we pass the comparison operators At Least `>=` and Less Than `<` as arguments to `analyze` to determine most-common or least-common bits. That leaves scope for extending the solution to other comparisons. 
But the problem here requires only most-common or least-common, which is At Least and its negation. 

The ternary conditional Cond is a compact way of expressing if/then/else. 
But everything else being equal, I prefer the Zen monks, who accept everything and act appropriately.

>*The Great Way is not difficult. It avoids only picking and choosing.* — [Daiyu Myokyo-ni Zenji](https://en.wikipedia.org/wiki/Myokyo-ni)

Instead of passing the operators as arguments, we could pass a 0 or 1 according to whether we want most-common or least-common bits.
It might also improve legibility to separate testing the filter results from the Do iteration. 

That leaves our complete solution:
```q
dg:"1"=read0`:day3.txt
a[`$"3-1"]:prd 2 sv'1 not\(sum dg)>=(count dg)%2
fltr:{x=(sum x)>=(count x)%2}                    / flag matches to most-common bit
filter:{x@$[count i:where z not/fltr y x;i;::]}  / filter rows
rating:{y first(til count y)filter[;;x]/flip y} 
OGCS:0 1                                         / O2 generator; CO2 scrubber
a[`$"3-2"]:prd 2 sv'OGCS rating\:dg
```

## [Day 4: Giant Squid](https://adventofcode.com/2021/day/4)

In the solving today’s problem we shall

* represent matrices as vectors, finding vectors more tractable
* work with a 3-dimensional array
* rather than looping and testing, we’ll do a classic array-language ‘overcompute’ and analyse the results

### Ingestion

The bingo boards are nicely readable in the text file, but will be more tractable as vectors.
```txt
14,30,18,8,3,10,77,4,48,67,28,38,63,43,62,12,68,88,54,32,17,21,83,64,97,53,24,2,60,96,86,23,20,93,65,34,45,46,42,49,71,9,61,16,31,1,29,40,59,87,95,41,39,27,6,25,19,58,80,81,50,79,73,15,70,37,92,94,7,55,85,98,5,84,99,26,66,57,82,75,22,89,74,36,11,76,56,33,13,72,35,78,47,91,51,44,69,0,90,52

13 62 38 10 41
93 59 60 74 75
79 18 57 90 28
56 76 34 96 84
78 42 69 14 19

96 38 62  8  7
78 50 53 29 81
88 45 34 58 52
33 76 13 54 68
59 95 10 80 63

36 26 74 29 55
...
```

Empty lines in the text file show us where to divide it into matrices.
```q
q)q:read0`:test4.txt
q)show nums:value first q
7 4 9 5 11 17 23 2 0 14 21 24 10 16 13 6 15 25 12 22 18 20 8 19 3 26 1
q)show boards:value each" "sv'(where 0=count each q)cut q
22 13 17 11 0  8  2  23 4  24 21 9 14 16 7  6  10 3  18 5 1  12 20 15 19
3  15 0  2  22 9  18 13 17 5  19 8 7  25 23 20 11 10 24 4 14 21 16 12 6
14 21 17 24 4  10 16 15 9  19 18 8 23 26 20 22 11 13 6  5 2  0  12 3  7
```

### Part 1

A perfectly sensible looping approach would follow real life. We would call each number and see if any board has won. 

We’re not going to do that. We’re going to call all the numbers and see where the wins occur.
```q
s:(or\')boards=/:\:nums  / states: call all the numbers in turn
```
The derived function `=/:\:` (Equal Each Right Each Left) gives us a Cartesian product on the Equal operator. The list `boards` is a matrix; list `nums` is a vector; the Cartesian product has three dimensions. Put another way, `boards` has rank 2; `nums` rank 1; so their Cartesian product has rank 3. 

The list `boards=/:\:nums` has an item for each board. Each item is a boolean matrix: the rows correspond to the called numbers, the columns to the board numbers. Here’s the first board: 1s flag the matches as they are called.
```q
q)first boards=/:\:nums
0000000000000010000000000b
0000000010000000000000000b
0000000000010000000000000b
0000000000000000000100000b
0001000000000000000000000b
0010000000000000000000000b
..
```
That 1 in the top row should correspond to the first number called.
```q
q)boards[0]where first first board=/:\:nums
,7
q)first nums
7
```
Bingo.

Of course, once a number is called, it stays called.
```q
q)(or\)first boards=/:\:nums
0000000000000010000000000b
0000000010000010000000000b
0000000010010010000000000b
0000000010010010000100000b
0001000010010010000100000b
0011000010010010000100000b
..
```
That is just the first board. We want the same for every board.
```q
s:(or\')boards=/:\:nums  / states: call all the numbers in turn
```
The transformation above is a useful pattern. 
Use the first item of a list to figure out what (function) to apply to an item  (here it is `(or\)`); then drop the `first` and Each the function. 

How to tell if a board has won? 
Here is the state of board 0 after nine numbers have been called. 
```q
q)s[0;9;]
0011101110011010000100000b
```
Has it won?
```q
q)5 cut s[0;9;]
00111b
01110b
01101b
00001b
00000b
```
No: that would require all 1s on at least one row. Or a column.
```q
q)any all each {x,flip x}5 cut s[0;9;]
0b
```
Now we can flag the wins.
```q
q)show w:{any all each {x,flip x} 5 cut x}''[s]
000000000000011111111111111b
000000000000001111111111111b
000000000001111111111111111b
```
From this we can see that the third board is the first to win and the second is 
the last to win. 
Also that they win on (respectively) the 11th and 14th numbers called. 
```q
q)sum each not w
13 14 11i
```
So the state of the winning board is `s[2;11;]`
```q
q)5 cut s[2;11;]
11111b
00010b
00100b
01001b
11001b
q)sum boards[2] where not s[2;11;] / sum the unmarked numbers
188
q)nums[11]*sum boards[2] where not s[2;11;] / board score
4512
```

### Part 2

For Part 2 we want the state of the losing board when it finally ‘wins’. That is board 1 after the 14th number.
```q
q)nums[14]*sum boards[1] where not s[1;14;]
1924
```
That makes our complete solution:
```q
q:read0`:day4.txt
nums:value first q
boards:value each" "sv'(where 0=count each q)cut q

s:(or\')boards=/:\:nums                             / states: call all the numbers in turn
w:sum each not{any all each b,flip b:5 cut x}''[s]  / flag wins
bs:{nums[x]*sum boards[y] where not s[y;x;]}        / score for board y after xth number
a[`$"4-1"]:bs . {m,x?m:min x} w                     / winning board score
a[`$"4-2"]:bs . {m,x?m:max x} w                     / losing board score
```
This is a nice example of the ‘overcomputing’ characteristic of vector languages. 
Clearly, an efficient algorithm could stop when it reached the first win. 
Or, for Part 2, the last win. 

But sometimes it is convenient to calculate all the results and search them. 
And, with vector operations, sometimes it is faster as well. 


## Day [5: Hydrothermal Venture](https://adventofcode.com/2021/day/5)

In solving today’s challenge, we

* write a simple function (ascending range) then wrap it to handle a larger domain
* use a test to index into a list of functions: equivalent to a case expression 
* index a string with a matrix to visualise the matrix

### Ingestion

Each vent is defined by two co-ordinate pairs.
```q
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
We’ll represent the vents as a list of 2×2 matrices.
```q
q)show vents:{value"(",ssr[;"->";";"]x,")"}each read0 `:test5.txt
0 9 5 9
8 0 0 8
9 4 3 4
2 2 2 1
7 0 7 4
6 4 2 0
0 9 2 9
3 4 1 4
0 0 8 8
5 5 8 2
```


### Part 1

```q
q)first vents
0 9
5 9
```
The second coords match so this is a horizontal line through
```q
0 9
1 9
2 9
3 9
4 9
5 9
```
which we could express as `(rng 0 5),'rng 9 9` if we had a function `rng` that returns an atom for `9 9`. We need only 
```q
q)range:{r:(::;reverse).[>]x;r{x+til y-x-1}. r x}
q)rng:{(0 1 .[=]x)first/range x}
q)rng each (5 9;5 3;9 9)  / some test cases
5 6 7 8 9
5 4 3
9
```
to get the points crossed by a vent.
```q
q)pts:{.[,']rng each flip x}
q)([]v:vents;p:pts each vents)
v       p
---------------------------------------------
0 9 5 9 (0 9;1 9;2 9;3 9;4 9;5 9)
8 0 0 8 (8 0;7 1;6 2;5 3;4 4;3 5;2 6;1 7;0 8)
9 4 3 4 (9 4;8 4;7 4;6 4;5 4;4 4;3 4)
2 2 2 1 (2 2;2 1)
7 0 7 4 (7 0;7 1;7 2;7 3;7 4)
6 4 2 0 (6 4;5 3;4 2;3 1;2 0)
0 9 2 9 (0 9;1 9;2 9)
3 4 1 4 (3 4;2 4;1 4)
0 0 8 8 (0 0;1 1;2 2;3 3;4 4;5 5;6 6;7 7;8 8)
5 5 8 2 (5 5;6 4;7 3;8 2)
```
We notice with satisfaction that `pts` finds the points for diagonal vents as well as vertical and horizontal ones. 

Find the points for just horizontal and vertical vents:
```q
q)vents where{any .[=]x}each vents
0 9 5 9
9 4 3 4
2 2 2 1
7 0 7 4
0 9 2 9
3 4 1 4

q)pts each vents where{any .[=]x}each vents
(0 9;1 9;2 9;3 9;4 9;5 9)
(9 4;8 4;7 4;6 4;5 4;4 4;3 4)
(2 2;2 1)
(7 0;7 1;7 2;7 3;7 4)
(0 9;1 9;2 9)
(3 4;2 4;1 4)

q)count each group raze pts each vents where{any .[=]x}each vents
0 9| 2
1 9| 2
2 9| 2
3 9| 1
4 9| 1
5 9| 1
9 4| 1
8 4| 1
7 4| 2
6 4| 1
5 4| 1
4 4| 1
3 4| 2
2 2| 1
2 1| 1
7 0| 1
7 1| 1
7 2| 1
7 3| 1
2 4| 1
1 4| 1
```
And count the points where vents overlap.
```q
q)plot:{count each group raze pts each x}
q)chp:{count where 1<x}  / count hot points
q)chp plot vents where {any .[=]x} each vents
5
```


### Part 2

The general case simply stops excluding the diagonal vents.
```q
q)chp plot vents
12
```

### Visualisation

We can also check our work against the map.
For this we’ll represent the 10×10 map as a 100-item vector and map the co-ordinate pairs into the range 0-99. 
```q
q)d:count each group 10 sv'raze pts each vents  / coords => (0-99)
9 | 2
19| 2
29| 2
39| 1
49| 1
59| 1
80| 1
..
```
Decompose the dictionary into key and value lists.
```q
q)(key;value)@\:d
9 19 29 39 49 59 80 71 62 53 44 35 26 17 8 94 84 74 64 54 34 22 21 70 72 73 4..
2 2  2  1  1  1  1  2  1  2  3  1  1  1  1 1  1  2  3  1  2  2  1  1  1  2  1..
```
Apply to these lists this projection of [Amend At](https://code.kx.com/q/ref/amend/) `@[100#0;;:;]`.
Its two omitted arguments make it a binary function.
We use [Apply](https://code.kx.com/q/ref/apply/) `.` to apply it to a list of its two arguments.
```q
q)@[100#0;;:;].(key;value)@\:d  / map to indices
1 0 0 0 0 0 0 0 1 2 0 1 0 0 1 0 0 1 0 2 1 1 2 0 1 0 1 0 0 2 0 1 0 1 2 1 0 0 0..
```
Putting that together:
```q
q)flip " 1234"10 cut @[100#0;;:;].(key;value)@\:count each group 10 sv'raze pts each vents
"1 1    11 "
" 111   2  "
"  2 1 111 "
"   1 2 2  "
" 112313211"
"   1 2    "
"  1   1   "
" 1     1  "
"1       1 "
"222111    "
```
That leaves our complete solution:
```q
vents:{value"(",ssr[;"->";";"]x,")"}each read0`:day5.txt
range:{r:(::;reverse).[>]x;r{x+til y-x-1}. r x}
rng:{(0 1 .[=]x)first/range x}                  / range: atom for .[=]x
pts:{.[,']range each flip x}                    / points of a vent
plot:{count each group raze pts each x}
chp:{count where 1<x}                           / count hot points

a[`$"5-1"]:chp plot vents where {any .[=]x} each vents
a[`$"5-2"]:chp plot vents
```


## [Day 6: Lanternfish](https://adventofcode.com/2021/day/6)

In solving today’s challenge we

* start with a naive strategy then get smarter
* use the Do iterator
* overcompute rather than iterate twice or write test logic

### Ingestion

```txt
2,3,1,3,4,4,1,5,2,3,1,1,4,5,5,3,5,5,4,1,2,1,1,1,1,1,1,4,1,1,1,4 ...
```
The first line of the text file is an integer vector. It does not get easier than this.

```q
lf:value first read0`:day6.txt
```


### Part 1

It’s tempting to model the lanternfish population as presented: as a vector of timer states, subtracting 1 on each day, resetting each 0 to 6 and appending an 8.
That lets us model progress day by day.
```q
q)lf:3 4 3 1 2  / lanternfish: test data
q)3{,[;sum[n]#8] (x-1)+7*n:x=0}\lf
3 4 3 1 2
2 3 2 0 1
1 2 1 6 0 8
0 1 0 5 6 7 8
```
And count them after 18 and 80 days.
```q
q)count 18{,[;sum[n]#8] (x-1)+7*n:x=0}/lf
26
q)count 80{,[;sum[n]#8] (x-1)+7*n:x=0}/lf
5934
```

### Part 2

But all this quickly gets out of hand. 
Each append to the vector entails making a copy.
Over 256 days the count exceeds 26 billion. 

A vector is an *ordered* list, but we do not need the lanternfish in any order. We need only represent how many fish have their timers in a given state. 
We could do this with a dictionary. 
```q
q)count each group lf
3| 2
4| 1
1| 1
2| 1
```
But even this is more information than we need. There are only nine possible timer values. 
A vector of nine integers will number the fish at each timer state.
```q
q)show lf:@[9#0;;1+]lf  / lanternfish school
0 1 1 2 1 0 0 0 0
```
Notice in the application of [Amend At](https://code.kx.com/q/ref/amend/) above the index vector `3 4 3 1 2` contains two 3s. 
The unary third argument of Amend At, `1+`, is applied twice at index 3. 
The iteration is implicit in Amend At and need not be specified. 

Now we can represent a day’s action with a `1 rotate`, which happily rotates the fish with expired timers round to position 8. 
But position 8 represents newly spawned fish: we also need to reset their parents’ timers by adding them at position 6.
```q
q)3 {@[1 rotate x;6;+;first x]}\ lf
0 1 1 2 1 0 0 0 0
1 1 2 1 0 0 0 0 0
1 2 1 0 0 0 1 0 1
2 1 0 0 0 1 1 1 1
```
After the required iterations, count the fish with `sum`.
```q
q)sum 256{@[1 rotate x;6;+;x 0]}/ lf
26984457539
```
Our complete solution:
```q
lf:@[9#0;;1+] value first read0`:day6.txt
a[`$("6-1";"6-2")]:sum each @[;80 256] 256{@[1 rotate x;6;+;first 0]}\lf
```
Above, rather than run the same iteration first 80 then 256 times with Over, we have run it 256 times with Scan, then selected the 80th and 256th state vectors from the result. 


## [Day 7: The Treachery of Whales](https://adventofcode.com/2021/day/7)

In solving today’s problem we

* represent a sparse array as a dictionary
* use the While iterator to implement a binary search of a solution space
* write a function that takes either a function or a list as one of its arguments

### Ingestion

```txt
1101,1,29,67,1102,0,1,65,1008,65,35,66,1005,66,28,1,67,65,20,4, ...
```
Our one-line text file has a thousand integers separated by commas. 
Still looks like a vector to q.
```q
q)value each read0`:day7.txt
1101 1 29 67 1102 0 1 65 1008 65 35 66 1005 66 28 1 67 65 20 4 0 1001 65 1 65..
```

### Part 1

We can represent crab positions as an integer vector.
```q
cp:16 1 2 0 4 2 7 1 2 14  / crab positions
```
The distance from `cp` to any position `x` is simply `abs cp-x`.
A naive solution calculates the fuel cost to all possible destinations.
```q
q)abs cp-\:til 1+max cp
16 15 14 13 12 11 10 9 8 7 6  5  4  3  2  1  0
1  0  1  2  3  4  5  6 7 8 9  10 11 12 13 14 15
2  1  0  1  2  3  4  5 6 7 8  9  10 11 12 13 14
0  1  2  3  4  5  6  7 8 9 10 11 12 13 14 15 16
4  3  2  1  0  1  2  3 4 5 6  7  8  9  10 11 12
2  1  0  1  2  3  4  5 6 7 8  9  10 11 12 13 14
7  6  5  4  3  2  1  0 1 2 3  4  5  6  7  8  9
1  0  1  2  3  4  5  6 7 8 9  10 11 12 13 14 15
2  1  0  1  2  3  4  5 6 7 8  9  10 11 12 13 14
14 13 12 11 10 9  8  7 6 5 4  3  2  1  0  1  2
```
Each column corresponds to a candidate destination.
```q
q)sum abs cp-\:til 1+max cp  / fuel costs
49 41 37 39 41 45 49 53 59 65 71 77 83 89 95 103 111
```
And finds the smallest.
```q
q)min sum abs cp-\:til 1+max cp
37
```
But we notice that the destination costs follow a smooth curve. 
Once we move past the optimum destination (2) the fuel costs rise steadily.
We could stop searching as soon as the fuel cost begins to rise again. 

A binary search of the solution space should converge quickly on the minimum, without calculating the fuel cost for every possible destination.

Moreover there are clusters of crabs in some positions.
We need only calculate the fuel cost for each distinct position, then weight it by the number of crabs there. 

Count the crabs at each position:
```q
q)show cd:count each group cp  / crab distribution
16| 1
1 | 2
2 | 3
0 | 1
4 | 1
7 | 1
14| 1
```
Calculate fuel cost for destination `y` in distribution `x`.
```q
q)fc1:{sum(value x)*abs y-key x}[cd;]  / fuel cost of destination (Part 1)
q)fc1 2  / fuel cost of moving to 2
37
```
Now we’ll use `fc1` to search the solution space of `til 1+max cd`.
Calculate the fuel cost at the halfway point and its neighbour. 
According to which cost is higher, drop half the solution space.

We start with a binary-search function. 
It evaluates a function at two adjacent values in the middle of a range and returns the upper or lower half of the range.
```q
q)show frd:(min;max)@\:cp  / full range of possible destinations
0 16
q)fc1 each 0 1+floor med frd
59 65
```
The gradient is ‘up’ so that’s a ‘go-left’ – we want the lower part of the range (0,16).
```q
q).[<]fc1 each 0 1+floor med frd
1b
```
If the midpoint is `m` we want `0,m`. Had the gradient been ‘down’, we’d want `m,16`.
```q
q)?[;r;m]1 not\.[<]fc1 each 0 1+m:floor med frd
0 8
```
The Zen monks in action again!
We use the result with [Vector Conditional](https://code.kx.com/q/ref/vector-conditional/) `?[;r;m]` to get one half of the range.

This can all be expressed as a binary-search function.
```q
q)bs:{?[;y;m]1 not\.[<]x each 0 1+m:floor med y}  / halve range y with fn x
q)bs[fc1] 0 16
0 8
```
We can wrap this with a ‘short list’ function that narrows the range until we are happy to evaluate every point in it: say, no more than 5 points.

We can use the [While](https://code.kx.com/q/ref/accumulators/#while) iterator with a simple test function `{neg[x]>.[-]y}[n;]`.
```q
q)sl:{[n;f;r]{neg[x]>.[-]y}[n;] bs[f;]/r}  / short list
q)sl[5;fc1;] 0 16
0 4
```
The last expression above could have been written `sl[5;fc1;0 16]`, but I wrote it as the unary projection of `sl[5;fc1;]` because the result `0 4` is a version of the third argument `0 16`, so writing it as a projection helps the reader see one range transformed into another range. (You might also think of the first two arguments as constraints, options, or parameters; and the third argument as ‘data’.) 
```q
q)min fc each rng sl[fc1;5;] 0 16
37
```
On my machine this is about 30× faster on the full puzzle data than the naive algorithm.


### Part 2

In Part 2 all that changes is the fuel-cost calculation.
Each distance has a multiplier.
```q
fm:sums til 1+max cp
fc2:{sum(value x)*fm abs y-key x}[cd;] / fuel cost of moving to position y
```
The difference between the two fuel-cost functions is so small it’s worth making it an argument.
```q
fc:{sum(value x)*y abs z-key x}[cd;;] / Part 2 fuel cost of moving to position z
```
This makes `fc` a binary; its first argument is the fuel multiplier. For Part 1 that can be the [Identity](https://code.kx.com/q/ref/identity/) operator `::`. 
Note here the use of an important q principle: functions, lists and dictionaries are all mappings from one domain to another. The syntax permits us to use either a function or a list as the first argument of `fc`.

Our complete solution:
```q
cd:count each group value first read0`:day7.txt  / crab distribution
frd:(min;max)@\:key cd  / full range of destinations
fc:{sum(value x)*y abs z-key x}[cd;;] / fuel cost of moving to position z
bs:{?[;y;m]1 not\.[<]x each 0 1+m:floor med y}  / halve range y with fn x
sl:{[f;n;r]{neg[x]>.[-]y}[n;] bs[f;]/r}  / short list
a[`$"7-1"]:min fc[::] each rng sl[fc[::];5;] frd
fm:sums til 1+max frd
a[`$"7-2"]:min fc[fm] each rng sl[fc[fm];5;] frd
```

The next article in this series will look at the problems in week 2 of the competition. 