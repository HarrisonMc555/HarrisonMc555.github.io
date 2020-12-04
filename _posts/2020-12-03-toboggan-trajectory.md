---
layout: post
title: "2020 Day 3: Toboggan Trajectory"
date: 2020-12-03 12:00:00 -0700
categories: coding adventofcode python
---

## Part 1

This post explores day 3 of Advent of Code 2020! You can find the description for part 1 of this exercise
[here](https://adventofcode.com/2020/day/3).


### The goal

For this exercise, we get a map of the trees in an area and need to figure out how many trees are on your path.


### Parsing the input

Let's start off by getting each line of input.

```python
def get_lines_from_file(filename):
    with open(filename) as f:
        return [line.strip() for line in f.readlines()]
```

And a test:

```python
>>> lines = get_lines_from_file('input.txt')
>>> lines[:2]
['.##.............##......#.....#', '.#.#................#..........']
```

Looks good so far. We could leave each line as a string, but that doesn't mean much. Let's transform each character into
an enum.

An [enum](https://en.wikipedia.org/wiki/Enumerated_type) is a type that has a finite set of values that it can
be. Instead of a number or a character (each of which can be many different values), an enum can only be one of the
types you list. In our example, we only have two possibilities: a tree (`#`) or an open space (`.`).

Python does not have enums as part of the language (unfortunately), but there is an
[enum](https://docs.python.org/3/library/enum.html) class that give us most of what we need.

```python
from enum import Enum, auto
class Tile(Enum):
    TREE = auto()
    OPEN = auto()
```

We also need some way of turning each character into a `Tile`. Let's add it to the `Tile` class itself. This would
normally be a good time for a switch statement, but, alas, Python doesn't have those either. Oh well.

```python
from enum import Enum, auto
class Tile(Enum):
    TREE = auto()
    OPEN = auto()

    def from_char(c):
        if c == '#':
            return Tile.TREE
        elif c == '.':
            return Tile.OPEN
        else:
            raise Exception(f'Invalid character {c}')
```

Normally I don't worry too much about error handling during Advent of Code puzzles because we have guarantees about the
file format. However, I don't want to accidentally get `None` (`null`) if I try to parse a newline character or
something, so we'll throw an exception if we don't get a `#` or `.`.

Finally, we'll just create a few helper functions that parse a single line or a group of lines.

```python
def parse_map(lines):
    return [parse_line(line) for line in lines]

def parse_line(line):
    return [Tile.from_char(c) for c in line]
```

I think we have enough pieces to get started.

```python
>>> lines = get_lines_from_file('input.txt')
>>> tree_map = parse_map(lines)
>>> print([row[:3] for row in tree_map[:3]])
>>> for row in tree_map[:2]:
...     print(row[:3])
...
[<Tile.OPEN: 2>, <Tile.TREE: 1>, <Tile.TREE: 1>]
[<Tile.OPEN: 2>, <Tile.TREE: 1>, <Tile.OPEN: 2>]
```

I think that looks right. I checked my input file and it matches! So what's next? Right, actually solving the
problem. Sometimes parsing is the hardest part.


### Counting the trees

Now we need to count all trees we're going to encounter by going on the slope right 3, down 1. Seems easy enough! The
one tricky part is that the map of trees repeats infinitely in the horizontal. Fortunately it doesn't repeat vertically,
or we'd never reach the bottom!

Let's with the simplest solution we can think of. We'll keep track of our current position both in the row and column
and keep track of the trees we hit. I'm using a few things here, including a two-dimensional array/list (`tree_map`),
comparison against an enum, and the `+=` operator. The `+=` is the add/assign operator, one of the "[in-place
operators](https://docs.python.org/3/library/operator.html#in-place-operators)" in Python (and many other
languages). The statement `x += y` is simply shorthand for `x = x + y`. Finally, the "height" of the map is the number
of rows, so once `row` is greater than the length of the `map` then we're off the edge vertically and we can stop.

With all that out of the way, let's look at our first pass.

```python
def count_trees(tree_map, slope_right, slope_down):
    row, column = 0, 0
    num_trees = 0
    while row < len(tree_map):
        if tree_map[row][column] == Tile.TREE:
            num_trees += 1
        row += slope_right
        column += slope_down
    return num_trees
```

This is really close, but it doesn't account for the horizontal repeating of the trees. If we try to run this, we'll get
the following error since we run off the edge of the map (horizontally).

```text
>>> count_trees(tree_map, 3, 1)
Traceback (most recent call last):
...
IndexError: list index out of range
```

Well that's what we expected at least. Let's try again.

How do we deal with an infinitely repeating map of trees? We don't have any computers with infinite space? Fortunately,
we don't actually have to store bigger and bigger versions of the map since they repeat every time. Instead, we can
simply wrap around and use the same map over and over again. The easiest way to do that is to simply let our `column`
value roll over every time it exceeds the width of the map. The modulus operator, `%`, is our friend here.

If you're unfamiliar with the modulus operator, it essentially returns the remainder after dividing two numbers. For
example, `7 ÷ 3 = 2 remainder 1`, so `7 % 3 = 1`. One of the great things about the modulus operator is that you can use
it to force a number between `0` and `n - 1`. As it turns out, those are exactly the valid indices into a list of length
`n`!

Here's our updated function:

```python
def count_trees(tree_map, slope_right, slope_down):
    row, column = 0, 0
    num_trees = 0
    num_columns = len(tree_map[0])
    while row < len(tree_map):
        if tree_map[row][column] == Tile.TREE:
            num_trees += 1
        row += slope_down
        column = (column + slope_right) % num_columns
    return num_trees
```

And that's about it! It's pretty common for coding solutions to be fairly straightforward once you set everything up
correctly. Here's hoping it works!

```python
>>> count_trees(tree_map, 3, 1)
176
```

And it works! Another reminder that everyone's inputs are different so everyone's solutions are different.


### Debugging

I was fortunate that my solution worked once I solved all of the obvious problems. If I spent more than a minute looking
at my code and it still wasn't producing the right answer, I would have tried my code against the example input. If I
still wasn't sure what was going on, I would add a `print` statement in the `while` loop to see what was going on as I
manually stepped through the example tree map.


## Part 2

Now, instead of just checking one slope, we need to check a bunch of slopes! Fortunately, that's not too much harder
than listing out all of the slopes to test. We'll use tuples this time and store the slopes as pairs of integers.

For your convenience, here is the list of slopes from [Advent of
Code](https://adventofcode.com/2020/day/3#part2). Remember that you have to solve part 1 before you can see part 2.

```text
Right 1, down 1.
Right 3, down 1. (This is the slope you already checked.)
Right 5, down 1.
Right 7, down 1.
Right 1, down 2.
```

And in Python:

```python
all_slopes = [(1, 1), (3, 1), (5, 1), (7, 1), (1, 2)]
```

Now we simply need to see how many trees we see for each slope and multiply them all together. There's no built-in
`product` method like the `sum` method, so let's write one really quick.

I'm taking advantage of the fact that if you multiply anything by `1` it stays the same, so if my final `result` starts
at `1` it won't change anything. Then I simply multiply each number in turn and keep tracking of the running total.

```python
def product(nums):
    result = 1
    for num in nums:
        result *= num
    return result
```

We should test this, though, before we get too far.

```python
>>> product([1, 2, 3])
6
>>> product([2, 2, 2])
8
>>> product([10, 3])
30
```

Great! Looks like it's working. Now we'll use another [list
comprehension](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions) to get the number of trees
for each slope.

```python
def all_trees_product(tree_map, slopes):
    num_trees = [count_trees(tree_map, right, down) for (right, down) in slopes]
    return product(num_trees)
```

Let's see how we did.

```python
>>> all_trees_product(tree_map, all_slopes)
5872458240
```

Wow, that's a big number. Fortunately, that big number is my answer! Fortunately, as long as part 1 was working
correctly, there's little chance of part 2 going too far astray.


## Conclusion

I hope you enjoyed this walk through how I solved Advent of Code 2020 day 3! This one wasn't too difficult, but I'm sure
it won't be long before they get pretty tricky.


### Mistakes I made

Some readers may be new to programming. I would hate for anyone to read these posts and think "I would never be able to
do that, I always make so many mistakes when programming!". It is completely normal to make all sorts of mistakes! Here
is a list of just a few of the mistakes I made while solving this problem.

  1. Forgetting to return `num_trees` at the end of `count_trees`
  2. Adding `slope_right` to `row` instead of `column` and vice versa
  3. Using `slope_right + slope_right` instead of `column + slope_right`
  4. Using square brackets instead of parentheses for the `len` in `len(tree_map[0])`
  5. Using `while column < len(tree_map)` instead of `while row < len(tree_map)`
