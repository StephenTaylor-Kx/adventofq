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


## Day 3: Binary Diagnostic

Ingestion here is a treat. 
Comparing the file lines to `"1"` gets us a boolean matrix.
```q
q)show dg:"1"=read0`:day3.txt / diagnostic
110011110101b
110011100010b
010100011010b
011001100000b
010011011101b
..
```

### Part 1

Finding the most common bit is just comparing the the sum of `dg` to half its count.
```q
q)sum[dg]>=count[dg]%2  / gamma bits
001110101101b
```
And the least-common bit is a slight variation.
```q
q)sum[dg]<count[dg]%2  / epsilon bits
110001010010b
```
which suggests finding both with a projection. 
```q
q)(>=;<){.[x](sum y;count[y]%2)}\:dg
001110101101b
110001010010b
```
Above, a comparison operator is passed to the lambda as its left argument.
Within the lambda, the [Apply operator](https://code.kx.com/q/ref/apply/) `.` is used to compare `sum y`
and `count[y]%2`. The [Each Left map iterator](https://code.kx.com/q/ref/maps#each-left-and-each-right) applies the lambda between the two compariuson operators and the diagnostic matrix. 

All that remains is to encode these two binary numbers as decimals and get their product. 
The complete solution:
```q
q)prd 2 sv'(>=;<){.[x](sum y;count[y]%2)}\:"1"=read0`:day3.txt
2967914
```

## Part 2

To find the oxygen generator rating we need to filter the rows of `dg` by an aggregation (most-common bit) of its first column, and so on. 
We are going to have to specify the iteration. 
Start by naming the aggregator we already defined.
```q
bc:{.[x](sum y;count[y]%2)}  / bit counter
```
Instead of working across the columns of `dg` we’ll flip it and iterate through its items (rows).
As each iteration will use the result of the previous iteration, we need the [Accumulator iterators](https://code.kx.com/ref/acumulators/). 
We’ll use the Scan iterator to filter the rows of `flip dg` until the next iteration would leave no rows left. 
Initially, all the rows are ‘in’.
```q
(til count dg){$[count i:x where test y x;i;x]}/flip dg
```
Here we can see the structure of the iteration. 
The initial state `til count dg` is a list of all the rows of`dg`.
The lambda being iterated tests the first column of `dg` and returns the rows that pass the test. 
Only the rows listed in the left argument are tested, so eliminated rows stay eliminated. 

Our `test` function will take a boolean vector and return the indexes that match the most-common bit. 

We can use the Scan iterator to watch the row indexes being filtered. 
```q
q)(til count dg) {$[count i:x where y[x]=bc[>=]y x;i;x]}\ flip"1"=read0`:test3.txt
1 2 3 4 7 8 9
2 3 4 8
2 3 4
2 3
,3
q)dg 3
10111b
```
That is the oxygen generator rating for the test data. 
Switching the comparison operator wil give us the CO2 scrubber rating.
So to be tidy, let’s make the comparison operator an argument of the lambda.
```q
q)(til count dg) {$[count i:x where y[x]=bc[z]y x;i;x]}[;;>=]\ flip"1"=read0`:test3.txt
1 2 3 4 7 8 9
2 3 4 8
2 3 4
2 3
,3
q)(til count dg) {$[count i:x where y[x]=bc[z]y x;i;x]}[;;<]\ flip"1"=read0`:test3.txt
0 5 6 10 11
5 11
,11
,11
,11
q)dg 11
01010b
```
And there’s the CO2 scrubber rating for the test data.
On to the puzzle data.
```q
q)dg:"1"=read0`:day3.txt
q)ogri:first(til count dg) {$[count i:x where y[x]=bc[z]y x;i;x]}[;;>=]/ flip dg / find O2 generator rating
q)csri:first(til count dg) {$[count i:x where y[x]=bc[z]y x;i;x]}[;;<]/ flip dg  / find CO2 scrubberr rating
q)dg ogri,csri
011110000111b
111001000110b
q)prd 2 sv'dg ogri,csri
7041258
```

## Day 4: Giant Squid

The bingo boards are nicely readable in the text file, but we shall find them more tractable as vectors.
```q
q)q:read0`:test4.txt
q)nums:value first q
q)show boards:value each" "sv'(where 0=count each q)cut q
22 13 17 11 0  8  2  23 4  24 21 9 14 16 7  6  10 3  18 5 1  12 20 15 19
3  15 0  2  22 9  18 13 17 5  19 8 7  25 23 20 11 10 24 4 14 21 16 12 6
14 21 17 24 4  10 16 15 9  19 18 8 23 26 20 22 11 13 6  5 2  0  12 3  7
```
A perfectly sensible looping approach would follow real life. We would call each number and see if any board has won. 

We’re not going to do that. We’re going to call all the numbers and see where the wins occur.
```q
s:(or\')boards=/:\:nums  / states: call all the numbers in turn
```
The derived function `=/:\:` (Equal Each Right Each Left) gives us a cross-product on the Equal operator. The list `boards=/:\:nums` has an item for each board. Each item is a boolean matrix: a row for each of the called numbers, the columns corresponding to the board numbers. Here’s the first board with 1s flagging the numbers as they are called in turn.
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
How to tell if a board has won? 
Here is the state of board 0 after 9 numbers have been called. 
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
Clearly not. That would require all 1s on at least one row. Or a column.
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
That makes our complete solution:
```q
q:read0`:test4.txt
nums:value first q
boards:value each" "sv'(where 0=count each q)cut q

s:(or\')boards=/:\:nums  / states: call all the numbers in turn
w:sum each not{any all each b,flip b:5 cut x}''[s]  / flag wins
bs:{nums[x]*sum boards[y] where not s[y;x;]} / score for board y after xth number
a[`$"4-1"]:bs . {m,x?m:min x} w / winning board score
a[`$"4-2"]:bs . {m,x?m:max x} w / losing board score
```
This is a nice example of the ‘overcomputing’ characteristic of vector languages. 
Clearly, an efficient algorithm would stop when it reached the first win. 
Or, for Part 2, the last win. 

But sometimes it is convenient to calculate all the results and search them. 
And, with vector operations, sometimes it is faster as well. 


## Day 5: Hydrothermal Venture

Each vent is defined by two co-ordinate pairs.
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
q)first vents
0 9
5 9
```


### Part 1

The second coords match so this is a horizontal line through
```q
0 9
1 9
2 9
3 9
4 9
5 9
```
which we could express as `(range 0 5),'range 9 9`. We need only 
```q
q)rng:{x+til y-x-1}  / ascending range
q)range:{@[;x](reverse rng reverse@;rng;first)2 sv(=;<).\:x}
q)range each (5 9;5 3;9 9)
5 6 7 8 9
5 4 3
9
```
to get the points crossed by a vent.

Note how `range` tests its argument pair for both equality and ascending, takes the resulting boolean pair as a decimal and indexes a list of three functions – equivalent to a case statement. 
But we can do better. If we move the test for equality into `rng`, both functions can be expressed with the ternary conditional [Cond](https://code.kx.com/q/ref/cond/).
```q
q)rng:{$[x=y;x;x+til y-x-1]}.  / range (not descending)
q)range:{$[.[>]x;reverse rng reverse x;rng x]}
q)pts:{.[,']range each flip x}
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
We can also check our work against the map.
For this we’ll represent the 10×10 map as a 100-item vector and map the co-ordinate pairs into the range 0-99. 
```q
q)flip " 1234"10 cut{@[100#0;key x;:;value x]}count each group 10 sv'raze pts each vents
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
vents:{value"(",ssr[;"->";";"]x,")"}each read0 `:day5.txt
rng:{$[x=y;x;x+til y-x-1]}.  / range (ascending only)
range:{$[.[>]x;reverse rng reverse x;rng x]}
pts:{.[,']range each flip x}  / points of a vent
plot:{count each group raze pts each x}
chp:{count where 1<x}  / count hot points

a[`$"5-1"]:chp plot vents where {any .[=]x} each vents
a[`$"5-2"]:chp plot vents
```


## Day 6: Lanternfish

### Part 1

It’s tempting to model the lanternfish population as a vector of timer states, subtracting 1 on each day, resetting each 0 to 6 and appending an 8.
That lets us model progress day by day.
```q
q)3{,[;sum[n]#8] (x-1)+7*n:x=0}\3 4 3 1 2
3 4 3 1 2
2 3 2 0 1
1 2 1 6 0 8
0 1 0 5 6 7 8
```
And count them after 18 and 80 days.
```q
q)count 18{,[;sum[n]#8] (x-1)+7*n:x=0}/3 4 3 1 2
26
q)count 80{,[;sum[n]#8] (x-1)+7*n:x=0}/3 4 3 1 2
5934
```

### Part 2

But all this gets out of hand over 256 days as the vector count exceeds 26 billion. 

A vector is an *ordered* list, but we do not need the lanternfish in any order. We need only represent how many fish have their timers in a given state. 
We could do this with a dictionary. 
```q
q)count each group 3 4 3 1 2
3| 2
4| 1
1| 1
2| 1
```
But even this is more information than we need. There are only nine possible timer values. 
A vector of nine integers will number the fish at each timer state.
```q
q)show lf:@[9#0;;1+]3 4 3 1 2  / lanternfish school
0 1 1 2 1 0 0 0 0
```
Notice in the application of [Amend At](https://code.kx.com/q/ref/amend/) above the index vector `3 4 3 1 2` contains two 3s. 
The unary third argument of Amend At, `1+`, is applied twice at index 3. 
The iteration is implicit in Amend At and need not be specified. 

Now we represent a day’s count-down with a `1 rotate`, which happily rotates the fish with expired timers round to position 8. 
But position 8 represents newly spawned fish. 
So we need also to reset their parents’ timers by adding them at position 6.
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
Above, rather than run the same iteration first 80 then 256 times, we have run it 256 times with Scan, then selected the 80th and 256th state vectors from the result.


## Day 7: The Treachery of Whales

We can represent crab positions as an integer vector.
```q
cp:16 1 2 0 4 2 7 1 2 14  / crab positions
```
The distance from `cp` to any position `x` is simply `abs cp-x`.
A brute-force solution calculates the fuel cost to all possible destinations.
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
q)fc1:{sum(value x)*abs y-key x}[cd;]  / Part 1 fuel cost of destination 
q)fc1 2  / fuel cost of moving to 2
37
```
Now we’ll use `fc1` to search the solution space of `til 1+max cd`.
Calculate the fuel cost at the halfway point and its neighbor. 
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
Note the use of `1 not\`. The form `1 f\x` is a handy functional equivalent of `(x;f x)`.
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
The last expression above could have been written `sl[5;fc1;0 16]`, but I wrote it as the unary projection of `sl[5;fc1;]` because the result `0 4` is a version of the third argument `0 16`, so writing it as a projection helps the reader see a range transformed into another range. (You might also think of the first two arguments as constraints, options, or parameters; and the third argument as ‘data’.) 
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