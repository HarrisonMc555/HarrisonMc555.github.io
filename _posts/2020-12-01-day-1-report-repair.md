---
layout: post
title: "2020 Day 1: Report Repair"
date: 2020-12-01 23:59:59 -0700
categories: coding adventofcode python
---

## Part 1

This post explores the first day of Advent of Code 2020! You can find the description for part 1 of this exercise [here](https://adventofcode.com/2020/day/1).


### The goal

This exercise tasks us with finding two numbers that sum to `2020` from our input (a list of numbers). Sounds easy enough!

### Reading the input

Every Advent of Code exercise gives you a text file that is your personalized input. This means that your solution will
be different than my solution, since we have different inputs.

My first goal for every exercise is typically to simply parse the input file. In this case, the input text file is a
list of numbers, one on each line.

We can use the following code to parse each line as a number and store it as a list.

```python
def get_nums(input_file):
    with open(input_file) as f:
        return [int(line.strip()) for line in f.readlines()]
```

Let's try it out!

```python
>>> nums = get_nums('input.txt')
>>> print(len(nums))
200
>>> print(nums[:10])
[1046, 1565, 1179, 1889, 1683, 1837, 1973, 1584, 1581, 192]
```

Looks like it's working!


### Exploring solutions

How can we find two numbers that sum to `2020`? It seems like we simply need to try all possible pairs of numbers and see
if their sum is `2020`. Let's try it!

Our code will simply contain two `for` loops (one for the first number and one for the second number) and a check to see
if they add up to the sum we want.

I'm going to put this code (and most future code) in functions. This allows us to use the same function for both test
inputs and the actual input and is generally a good practice to follow.

```python
def find_pair_that_adds_to(nums, total):
    for num1 in nums:
        for num2 in nums:
            if num1 + num2 == total:
                return num1, num2
```

The exercise provides us with an example that is much smaller than our actual input. Let's try our code on that example
first and see if it works.

```python
>>> example_nums = [1721, 979, 366, 299, 675, 1456]
>>> find_pair_that_adds_to(example_nums, 2020)
(1721, 299)
```

It worked! Let's see if it works on the bigger input.

```python
>>> nums = get_nums('input.txt')
>>> find_pair_that_adds_to(nums, 2020)
(1373, 647)
```

Hooray! We can check, and, sure enough, `1373 + 647 = 2020`. Keep in mind that this is the answer for my input; yours
will be different.

The final part of the exercise is to simply multiply the two numbers to get our final result. Our final solution looks
something like this:

```python
>>> nums = get_nums('input.txt')
>>> x, y = find_pair_that_adds_to(nums, 2020)
>>> x * y
888331
```


## Part 2

Part 2 of each exercise is hidden until you solve part 1. If you don't want spoilers, stop now! Otherwise I'll give a
brief description of part 2 and explain how I approached it.


### The goal

Things are getting more interesting. Instead of finding _two_ numbers that add up to `2020`, now we need to find _three_!


### Exploring solutions

Let's try using the same strategy as we did last time, just with three numbers instead of two.

```python
def find_triplet_that_adds_to(nums, total):
    for num1 in nums:
        for num2 in nums:
            for num3 in nums:
                if num1 + num2 + num3 == total:
                    return num1, num2, num3
```

Let's try it with the example numbers:

```python
>>> find_triplet_that_adds_to(example_nums, 2020)
(979, 366, 675)
```

Looks like it worked, since `979 + 366 + 675 = 2020`. Let's try it for the real input!

```python
>>> find_triplet_that_adds_to(nums, 2020)
(511, 195, 1314)
```

It worked! `511 + 195 + 1314 = 2020`. Our final solution for part 2 looks something like this:

```python
>>> nums = get_nums('input.txt')
>>> x, y, z = find_triplet_that_adds_to(nums, 2020)
>>> x * y * z
130933530
```


## Conclusion

And that's it for day 1! I hope you enjoyed that brief walk through how I approach a day of Advent of Code.

There are some ways in which we could improve our solution. If that sounds interesting to you, read on! Otherwise feel
free to move on to day 2.


## Improving our solution

The solution we have created so far is sufficient to solve this exercise, even in the worst case. However, as it turns
out, Eric Wastl was being really nice to us in this exercise. If we look at the length of the input list, there are only
200 numbers.

```python
>>> nums = get_nums('input.txt')
>>> len(nums)
200
```

Trying all possible combinations of three numbers among a list of `200` numbers may sound like a lot, but for a computer,
it's actually not that bad. There are `8,000,000` (8 million) possible combinations of three numbers (without any logic to
avoid repeated combinations).

However, if the input list had `300` numbers, there would be `27,000,000` (27 million) combinations. If there had been
`1,000` numbers, there would have been `1,000,000,000` (1 billion) combinations. As you can see, the number of
combinations grows rapidly as the input list grows longer.

As it turns out, there are a few ways to drastically reduce the number of combinations we need to explore.


### Improved solution #1: Walking from the ends

Let's return to part 1 and see if there is a faster way to find a pair of numbers that sum to a given number. Let's look
at the list of example numbers again.

```python
>>> example_nums
[1721, 979, 366, 299, 675, 1456]
```

Let's try sorting the list and seeing if anything sticks out to us.

```python
>>> sorted_example_nums = list(sorted(example_nums))
>>> sorted_example_nums
[299, 366, 675, 979, 1456, 1721]
```

If we look at the first and last number, we see that they add up to `2020`. Since that was too easy, let's choose a new
sum to look for. Let's try `1654` (which is `675 + 979`).

If we try the first and last (smallest and biggest) number, then we get `299 + 1721 = 2020`. That sum is too big, since
`2020 > 1654`. If we think about it, we can realize that there is no way to sum to `1654` if one of the numbers is
`1721`. If one of the numbers were `1721`, then the sum will always be too big! So, we can confidently conclude that
both numbers must be less than `1721`. To get our next candidate pair, let's keep `299` but consider the _next_ largest
number, `1456`.

With thse numbers we have `299 + 1456 = 1755`, which is once again too large. If we move the larger number down again,
we get `299 + 979 = 1278`. Now, our number is too small, since `1278 < 1654`! If we use the same logic as last time (but
in reverse), we can determine that the smaller number must be larger than `299`. So, we can replace `299` with the next
smallest number.

Our next pair is `366 + 979 = 1345`, which is still too small (`1345 < 1654`). The next pair is `675 + 979 = 1654`. We
have now found our pair!

I'm not aware of a name for this algorithm, so I'll just call it "walking from the ends". The great thing about this
solution is that it only requires going through the list once, instead of going through the list once _for each
element_. It does require sorting the input list, though. If you're familiar with big-O notation, this solution is O(n
log(n)) instead of O(n<sup>2</sup>).

Here is an implementation of the "walking from the ends" algorithm.

```python
def find_pair_that_adds_to(sorted_nums, total):
    index_low = 0
    index_high = len(sorted_nums) - 1
    while index_low < index_high:
        low = sorted_nums[index_low]
        high = sorted_nums[index_high]
        cur_total = low + high
        if cur_total < total:
            index_low += 1
        elif cur_total > total:
            index_high -= 1
        else:
            return (low, high)
```

However, this is only an implementation for finding a pair. What about finding a triplet?

I'm unaware of anyway to improve the search besides simply iterating through every element and searching for a pair that
sums to the desired sum minus the current number. Here is an example implementation:

```python
def find_triplet_that_adds_to(sorted_nums, total):
    for num1 in sorted_nums:
        tup = find_pair_that_adds_to(sorted_nums, total - num1)
        if tup:
            num2, num3 = tup
            return num1, num2, num3
```

The total time complexity for this algorithm is O(n<sup>2</sup>). The sorting is O(n log(n)) but is only performed
once. The triplet algorithm is an O(n<sup>2</sup>) algorithm, which trumps the O(n log(n)) complexity.

Note that this could potentially use the same number more than once. If that is unacceptable, you can modify the logic
in `find_pair_that_adds_to` to not consider a given index.


### Improved solution #2: Hashing

You can also hash each number into a set/dictionary. This allows you to test, in constant time, whether a number appears
in your list. This allows you to find a pair of elements that sums to a given number in linear, or O(n), time, without
needing to sort the list first. Finding a triplet of elements can be done in quadratic time, or O(n<sup>2</sup>), just
like the "walking from the ends" algorithm.

Here is an example implementation:

```python
def find_triplet_that_adds_to(nums, total):
    s = set(nums)
    for i, x in enumerate(nums):
        for y in nums[i:]:
            z = total - x - y
            if z in s:
                return x, y, z
```

Once again, this could lead to using a value more than once. Using a dictionary/counter that stores the number of
occurrences would be a simple way to ensure that each value is only used once.
