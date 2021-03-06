# Disorderly Escape

- [Disorderly Escape](#disorderly-escape)
  - [Problem](#problem)
  - [Test cases](#test-cases)
  - [Brute Force Approach](#brute-force-approach)
    - [Analysis](#analysis)
    - [Solution #1 (Brute Force Approach)](#solution-1-brute-force-approach)
  - [Mathematical Approach](#mathematical-approach)
    - [Group Theory](#group-theory)
    - [Burnside Counting Theorum](#burnside-counting-theorum)
    - [Polya Enumeration Theorum](#polya-enumeration-theorum)
    - [Calculating the Cycle Index](#calculating-the-cycle-index)
    - [Putting it all Together](#putting-it-all-together)
    - [Solution #2](#solution-2)
  - [Bibliography](#bibliography)

> *This puzzle came from the Google FooBar site where it was a level 5 problem. Please note that I still do not fully understand the theoretical details involved in this problem. Since finding this problem on Google FooBar, I have been somewhat obsessed with understanding the maths involved, and this page is my poor attempt at documentating the research I've carried out. I'm not a trained mathematician, and the information below should not be relied on as being mathematically sound. Please let me know if I've explained anything incorrectly.*

## Problem

Oh no! You've managed to free the bunny prisoners and escape Commander Lambdas exploding space station, but her team of elite starfighters has flanked your ship. If you dont jump to hyperspace, and fast, youll be shot out of the sky!

Problem is, to avoid detection by galactic law enforcement, Commander Lambda planted her space station in the middle of a quasar quantum flux field. In order to make the jump to hyperspace, you need to know the configuration of celestial bodies in the quadrant you plan to jump through. In order to do _that_, you need to figure out how many configurations each quadrant could possibly have, so that you can pick the optimal quadrant through which youll make your jump.

There's something important to note about quasar quantum flux fields' configurations: when drawn on a star grid, configurations are considered equivalent by grouping rather than by order. That is, for a given set of configurations, if you exchange the position of any two columns or any two rows some number of times, youll find that all of those configurations are equivalent in that way - in grouping, rather than order.

Write a function solution(w, h, s) that takes 3 integers and returns the number of unique, non-equivalent configurations that can be found on a star grid w blocks wide and h blocks tall where each celestial body has s possible states. Equivalency is defined as above: any two star grids with each celestial body in the same state where the actual order of the rows and columns do not matter (and can thus be freely swapped around). Star grid standardization means that the width and height of the grid will always be between 1 and 12, inclusive. And while there are a variety of celestial bodies in each grid, the number of states of those bodies is between 2 and 20, inclusive. The solution can be over 20 digits long, so return it as a decimal string. The intermediate values can also be large, so you will likely need to use at least 64-bit integers.

For example, consider w=2, h=2, s=2. We have a 2x2 grid where each celestial body is either in state 0 (for instance, silent) or state 1 (for instance, noisy). We can examine which grids are equivalent by swapping rows and columns.

```
00
00
```

In the above configuration, all celestial bodies are "silent" - that is, they have a state of 0 - so any swap of row or column would keep it in the same state.

```
00 00 01 10
01 10 00 00
```

1 celestial body is emitting noise - that is, has a state of 1 - so swapping rows and columns can put it in any of the 4 positions. All four of the above configurations are equivalent.

```
00 11
11 00
```

2 celestial bodies are emitting noise side-by-side. Swapping columns leaves them unchanged, and swapping rows simply moves them between the top and bottom. In both, the _groupings_ are the same: one row with two bodies in state 0, one row with two bodies in state 1, and two columns with one of each state.

```
01 10
01 10
```

2 noisy celestial bodies adjacent vertically. This is symmetric to the side-by-side case, but it is different because there's no way to transpose the grid.

```
01 10
10 01
```

2 noisy celestial bodies diagonally. Both have 2 rows and 2 columns that have one of each state, so they are equivalent to each other.

```
01 10 11 11
11 11 01 10
```

3 noisy celestial bodies, similar to the case where only one of four is noisy.

```
11
11
```

4 noisy celestial bodies.

There are 7 distinct, non-equivalent grids in total, so solution(2, 2, 2) would return 7.

## Test cases

```
Input:
solution.solution(2, 3, 4)
Output:
430

Input:
solution.solution(2, 2, 2)
Output:
7
```
## Brute Force Approach

### Analysis

Although it turns out that this problem ultimately requires a good understanding of group theory and combinatorics to solve efficiently, it is possible to solve this using a relatively simple brute-force algorithm, albeit for the smallest grids only:

### Solution #1 (Brute Force Approach)

```python
import numpy as np
import itertools

class State:
    """
    Represents a single coloring / state of a (w x h) grid.
    """

    def __init__(self, state):
        self.state = state
    
    def __str__(self):
        """
        Pretty-prints a state.
        """
        out = "\n".join(["".join([str(x) for x in row]) for row in self.state])
        return out

    
    def equals(self, state2):
        """Returns true if current state equals another state."""
        if len(self.state) != len(state2.state):
            return False

        for i, r in enumerate(self.state):
            for j, c in enumerate(self.state[i]):
                if c != state2.state[i][j]:
                    return False
        return True

    def transform(self, t):
        """
        Transforms a state into another state using a transform.
        """
        result = []
        for row in t:
            r = []
            for item in row:
                row = int(item[0])
                col = int(item[1])
                r.append(self.state[row][col])
            result.append(r)
        return State(result)

class Grid:
    """
    Represents a (w x h x s) grid.
    """
    
    def __init__(self, w, h, s):
        self.w = w  # width
        self.h = h  # height
        self.s = s  # states (same as colors)
        
        # Total possible permutations
        self.count = self.s ** (self.w * self.h)

        
    @staticmethod
    def __numberToBase(n, b):
        """Converts a number to base, b."""
        if n == 0:
            return [0]
        digits = []
        while n:
            digits.append(int(n % b))
            n //= b
        return digits[::-1]

    def states(self):
        """
        Gets all state permutations.
        """
        states = []
        for i in range(self.count):
            leftpad = [0] * (self.w * self.h)
            value = leftpad + Grid.__numberToBase(i, self.s)
            value = value[-(self.w * self.h):]

            # convert to 2d matrix
            grid = np.reshape(value, (-1, self.w)).tolist()
            states.append(State(grid))
            
        return states

    def transformations(self):
        """
        Gets all possible transformations on the grid. Rows + columns can be arranged in any order.
        Therefore there are (w! x h!) transformations.
        """
        numbers = "0123456789"

        transformations = []
        for rowPerm in itertools.permutations(numbers[:self.h]):
            for colPerm in itertools.permutations(numbers[:self.w]):
                transform = []
                for r in rowPerm:
                    for c in colPerm:
                        action = [r,c]
                        transform.append(action)

                transformations.append(np.reshape(transform, (self.h, self.w, 2)).tolist())

        return transformations

    def classes(self):
        """
        Returns the unique classes of states.
        """
        classes = [] # each class contains an array of similar states
        transforms = self.transformations()
        
        for state in self.states():
            found = False
            for cls in classes:
                testState = cls[0] # used to compare
                for transform in transforms:
                    newState = state.transform(transform)
                    if newState.equals(testState):
                        cls.append(state)
                        found = True
                        break

            if not found:
                # if got here, then no existing classes match - create new
                classes.append([state])

        return classes

    def solve(self):
        """
        Returns the number of classes of states in a (w x h x s) grid.
        """
        return len(self.classes())

    def pretty(self):
        """
        Prints the unique classes, with list of permutations in each class.
        """
        for cls in self.classes():
            print("=" * self.w)
            print(f"\n{' ' * self.w}\n".join([str(grid) for grid in cls]))
            
        
# Test Cases

assert Grid(3,1,2).solve() == 4
assert Grid(3,1,3).solve() == 10
assert Grid(3,1,4).solve() == 20

assert Grid(4,1,2).solve() == 5
assert Grid(4,1,3).solve() == 15
assert Grid(4,1,4).solve() == 35

assert Grid(5,1,2).solve() == 6
assert Grid(5,1,3).solve() == 21
assert Grid(5,1,4).solve() == 56

assert Grid(2,3,4).solve() == 430
assert Grid(2,2,2).solve() == 7
Grid(2,2,2).pretty()
```

This algorithm works by generating every single permutation of the grid of dimension (w x h s), then calculating how many unique configurations there are by taking each permutation in turn, and applying each of the transformations in turn, to see if the resulting configuration has been 'seen' before. As you can imagine, this algorithm is incredibly inefficient, and only works for the smallest test cases. It does however work for the 2 provided test cases, and passes both. To solve the problem in a more powerful way, we need to resort to some mathematics theory.

## Mathematical Approach

In order to solve this problem for non-trivial use-cases, a mathematical approach is required. This involved some research on my part.

### Group Theory

The area of mathematics useful to us here is known as 'Group Theory' (https://en.wikipedia.org/wiki/Group_theory). As usual, with most things mathematical, there are a number of definitions and symbols which are required to be understood first. A good summary of all the mathematical notation can be found at:

https://en.wikipedia.org/wiki/List_of_mathematical_symbols_by_subject

The following can be determined from the question:

- Each cellestial body (w x h x s) can be viewed as an element of a set, `S`
- Given dimensions of (w x h x s), there are `s^(wh)` unique permutations. Each permutation can be thought of as a unique 'colouring' with each grid location being coloured by one of `s` colours.
- The question defines rules that define how 2 or more configurations can be deemed to be equivalent (i.e. by swapping rows and/or columns). The transformation processes of swapping rows and/or columns) constitute a group.
- A group can be viewed as a bunch of actions that apply to a set
- The problem requires us to calculate the number of 'equivalent' configurations (taking into account the group transformations). These are know as equivalence classes or 'orbits'.

### Burnside Counting Theorum

The Burnside Lemma (or Burnside Counting Theorum) states that the number of orbits is equal to the average number of points fixed by an element of the group, G:

https://en.wikipedia.org/wiki/Burnside%27s_lemma

We can apply the Burnside Counting Theorum to the 2x2x2 test case. This test case contains a 2x2 grid. To assist describing the maths, I'll label the elements in this grid as follows:

```
---------
| a | b |
---------
| c | d |
---------
```

- The 2x2 grid has 4 elements in it. Each element can have 2 states (0 or 1)
- There are 4 ways to transform the grid (hence 4 group members):
  - We can leave all 4 grid elements unchanged (the identity)
  - We can swap the rows (a + b swap with c + d)
  - We can swap the columns (a + c swap with b + d)
  - We can swap both rows and columns together (a + d swap, b + c swap)

We now need to count the number of permuations that would be fixed (remain unchanged) after each group element is applied:


| g         | Description                                                                                                                                            | Fix(g)     |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------- |
| e         | Identity (Does nothing). All original set members are fixed.                                                                                           | 2 ^ 4 = 16 |
| row       | If swapping the rows, then b + d must be the same as a + c for the colouring to remain unchanged. Therefore we can choose any combination of b + d.    | 2 ^ 2 = 4  |
| col       | If swapping the columns, then c + d must be the same as a + b for the colouring to remain unchanged. Therefore we can choose any combination of a + b. | 2 ^ 2 = 4  |
| col + row | If swapping both columns and rows, then a + d must be the same colour, as do b + c. Therefore we are free to choose any combination of 2 positions.    | 2 ^ 2 = 4  |

The total number of fixed set members is 16 + 4 + 4 + 4 = 28

Dividing by the group size, we get 28 / 4 = 7. This is the correct answer.

### Polya Enumeration Theorum

With the above 2x2x2 grid, it can be noticed that some permutations appear more than others. For example, the grids where all colours are the same (all 0's or all 1's) only appear once:

```
---------
| 1 | 1 |
---------
| 1 | 1 |
---------

or

---------
| 0 | 0 |
---------
| 0 | 0 |
---------
```

However, the grid with 3 zeros and a single '1' occur 4 times:

```
---------   ---------   ---------   ---------
| 1 | 0 |   | 0 | 1 |   | 0 | 0 |   | 0 | 0 |
---------   ---------   ---------   ---------
| 0 | 0 |   | 0 | 0 |   | 1 | 0 |   | 0 | 1 |
---------   ---------   ---------   ---------
```

Polya captured the reason behind this and found an elegant way to describe it. The key to solving the problem is to study the 'cycle structure' of the permutations.

For example, if a set `123456` is transformed to `654321` (e.g. reflection) we can write the transform as 3 2-cycles:

```
s = (16)(25)(34)
```
This means:
- Move 1 to 6, and 6 back to 1
- Move 2 to 5, and 5 back to 2
- Move 3 to 4, and 4 back to 3

We represent this cycle by writing:

<img src="https://latex.codecogs.com/svg.latex?\Large&space;x_2^3" />

This can be read as '3 cycles of length 2'.

Another example (although not relevant for this question), is describing the cycle index for rotating the 2x2 grid 90 degrees. For example:

```
---------        ---------
| 1 | 2 |        | 4 | 1 |
---------   =>   ---------
| 4 | 3 |        | 3 | 2 |
---------        ---------
```

This cycle would be written as (1234) and read as:

`Move 1 to 2, 2 to 3, 3 to 4, and 4 back to 1`

For the identity, we have 4 cycles each of length 1, which keep all the elements where they are:

(1)(2)(3)(4)

The important thing to note is that a permutation / action has no effect on a colouring if (and only if) each cycle consists of set elements of the same colour.

Therefore, using the reflection example of the set of 6 elements above (with 3 cycles of length 2), then if there are 2 colour states for each position, there are:

2^3 = 8 ways to colour 6 elements so that the reflection transformation described above has no effect.

In general, if G is a permutation group acting on set, X, then the cycle polynomial of G is:

<img src="https://latex.codecogs.com/gif.latex?\bg_white&space;P_{(G,X)}(x_1,&space;...,&space;x_n)&space;=&space;\frac{1}{G}\sum_{g\in&space;G}&space;x_1^{j_1(g)}&space;...&space;x_n^{j_n(g)}" />

where `j_k(g)` is the number of cycles of length `k` in the permutation `g`.

Using this on our original 2x2x2 problem, and counting cycles:

| g         | Cycles       | Fix(g)     |
| --------- | ------------ | ---------- |
| e         | (a)(b)(c)(d) | 2 ^ 4 = 16 |
| row       | (ac)(bd)     | 2 ^ 2 = 4  |
| col       | (ab)(cd)     | 2 ^ 2 = 4  |
| col + row | (ad)(bc)     | 2 ^ 2 = 4  |

### Calculating the Cycle Index

We need a way to generate the correct cycle index coefficients. If we have a set of 4 elements {1,2,3,4}, and each can take one of 2 values (0/1), we know there are 4! = 24 ways of arranging the 4 elements. Therefore there are 24 group actions (including the identity) which can act on the 4 members:

https://groupprops.subwiki.org/wiki/Symmetric_group:S4

| Partition(s)  | Elements                                                                       | Term                                                |
| ------------- | ------------------------------------------------------------------------------ | --------------------------------------------------- |
| 4             | (1234), (1243), (1324), (1342), (1423), (1432)                                 | 6x<sub>4</sub><sup>1</sup>                          |
| 3 + 1         | (123)(4), (132)(4), (234)(1), (243)(1), (341)(2), (314)(2), (412)(3), (421)(3) | 8x<sub>3</sub><sup>1</sup>x<sub>1</sub><sup>1</sup> |
| 2 + 2         | (1,2)(3,4), (1,3)(2,4), (1,4)(2,3)                                             | 3x<sub>2</sub><sup>2</sup>                          |
| 2 + 1 + 1     | (1,2)(3)(4), (1,3)(2)(4), (1,4)(2)(3), (2,3)(1)(4), (2,4)(1)(3), (3,4)(1)(2)   | 6x<sub>2</sub><sup>1</sup>x<sub>1</sub><sup>2</sup> |
| 1 + 1 + 1 + 1 | (1)(2)(3)(4)                                                                   | 1x<sub>1</sub><sup>4</sup>                          |

Note that the sum of the coefficients of the terms is also 24. The cycle index is the result of combining all the above terms into a single polynomial expression. This polynomial describes how a group of permutations act on a set. If we know the cycle index polynomial, we can easily calculate the number of equivalence classes thus solving the problem.

### Putting it all Together

For a given w and h, we create 2 nested loops, iterating through every combination of partitions for size w and h respectively. This will let us iterate through every combination of cycle lengths for the given grid.

For example, if w is 3 and h is 5, then we cross calculate every cross-combination of:

```
[3]
[2,1]
[1,1,1]

with

[5]
[4,1]
[3,2]
[3,1,1]
[2,2,1]
[2,1,1,1]
[1,1,1,1,1]
```

For each combination of cycle length for w and h, we calculate the cycle counts for the corresponding cycleW and cycleH inputs. The cycle count is then used to calculate the number of permutations fixed by the 2 partition groups using:

```
total += cc * (s**sum([sum([gcd(i, j) for i in cycleW]) for j in cycleH]))
```

When calculating the number of fixed permutations, we raise the number of states (s) to the number of element fixed in the grid - this is calculated by iterating through each of the partitions (cycleW and cycleH), and calculating the lowest common denominator - this gives us the size of the cycle common to both.

Finally, once we loop through all the partition combinations, summing the total number of permutations fixed by the all the groups, the total is averaged by dividing by the number of elements in the group, which in our case is `w! x h!`.

### Solution #2

A final solution could look something as follows (See bibliography for source articles that heavily assisted this solution):

``` python
from collections import Counter
from math import gcd

def fact(n):
    """
    Calculates factorial of a number n
    """
    if n == 0:
        return 1
    else:
        return n * fact(n-1)

def part(n, m = None):
    """
    Generates partitions for an integer n, with maximum summand, m.
    """
    results = []
    if m == None:
        m = n
    if n == 1:
        results.append([1])
        return results
    elif m == 1:
        results.append([1] * n)
        return results
    else:
        for i in range(m, 0, -1):
            if i == n:
                results.append([i])
            elif i < n:
                for result in part(n - i, i):
                    results.append([i] + result)
        return results

def cycleCount(partition):
    """
    Gets the cycle count coefficient for a particular partition set. This can be used to build up the cycle index.
    """
    total = sum(partition)
    cycle_count = fact(total)
    for a, b in Counter(partition).items():
        cycle_count //= (a**b) * fact(b)
        
    return cycle_count

def solution(w, h, s):
    """
    Main entry point.
    """
    total = 0
    for cycleW in part(w):
        for cycleH in part(h):
            cc = cycleCount(cycleW) * cycleCount(cycleH)
            total += cc * (s**sum([sum([gcd(i, j) for i in cycleW]) for j in cycleH]))
    
    result = total // (fact(w) * fact(h))
    return result

# Tests

assert fact(0) == 1
assert fact(1) == 1
assert fact(2) == 2
assert fact(3) == 6
assert fact(4) == 24
assert fact(5) == 120

assert cycleCount([4]) == 6
assert cycleCount([3,1]) == 8
assert cycleCount([2,2]) == 3
assert cycleCount([2,1,1]) == 6
assert cycleCount([1,1,1,1]) == 1

assert solution(2,2,2) == 7
assert solution(2,3,4) == 430

assert solution(4,4,20) == 1137863754106723400
assert solution(5,5,20) == 23301834615661488487765745000
assert solution(6,6,20) == 132560781153101038829213988789736592649360
assert solution(7,7,20) == 221619886894198821201872678876163305792210161226545392840
assert solution(8,8,20) == 113469378614817897312718329989374518983724697432844009920312263602471667640
```

## Bibliography

- https://en.wikipedia.org/wiki/Cycle_index
- https://bogobogosort.wordpress.com/2018/03/30/completing-google-foobar/
- https://groupprops.subwiki.org/wiki/Symmetric_group:S4
- https://stackoverflow.com/questions/61689832/disorderly-escape-google-foobar-2020-not-passing-test-cases
- https://www.sciencedirect.com/science/article/pii/0012365X9390015L