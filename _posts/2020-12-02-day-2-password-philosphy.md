---
layout: post
title: "2020 Day 2: Password Philosophy"
date: 2020-12-01 12:00:00 -0700
categories: coding adventofcode python
---

## Part 1

This post explores day 2 of Advent of Code 2020! You can find the description for part 1 of this exercise
[here](https://adventofcode.com/2020/day/2).


### The goal

For this exercise, we get a list of passwords and need to determine which ones are valid. However, instead of having the
same rules apply to all passwords, each password entry has additional information that determines what constitutes a
valid password.

Here is the example list of passwords:

```text
1-3 a: abcde
1-3 b: cdefg
2-9 c: ccccccccc
```

The numbers specify the minimum and maximum number of times the given letter can appear.


### Reading the input

First, let's start by reading the lines from the input file.

```python
def get_lines_from_file(filename):
    with open(filename) as f:
        return [line.strip() for line in f.readlines()]
```

Next, we need to parse each line. Each line has four different pieces of information that we care about:

  1. The minimum number
  2. The maximum number
  3. The letter
  4. The password

We could use Python's `.split()` method to split on spaces, hyphens, and colons. However, this is a perfect time to use
a [regular expression](https://www.regular-expressions.info/)! Regular expressions are perfect when the input is in
consistent, simple format.

Let's explore the format of each line a little closer.

```text
 +---------- hyphen
 |
 | +-------- space
 | |
 | | +------ colon
 | | |
 | | |   +-- * password
 | | |   |
 | | | /---\
1-3 a: abcde
| | | |
| | | +----- space
| | |
| | +------- * letter
| |
| +--------- * minimum number
|
+----------- * maximum number
```

Every part of the format is important when creating a regular expression, since a mismatch will prevent the pattern from
matching. Each section labeled with a `*` is a piece that we want to keep and remember. The other parts are just for
formatting and can be thrown away once we get the pieces we need.

We want to create a regular expression pattern that matches this pattern and remembers the parts we care about. The
pieces we care about need to be in groups (surrounded by parentheses) so we can extract them later. This is the pattern
I came up with. If you're unfamiliar with regular expressions don't worry too much about it. They are a good tool,
though, so I would recommend learning about them at some point!

```text
(\d+)-(\d+) ([a-z]): ([a-z]+)
```

Fortunately for us, Python includes an implementation of regular expressions by default with the `re` module. We can
"compile" the regular expression into a pattern that we can use repeatedly like so:

```python
import re
pattern = re.compile('(\d+)-(\d+) ([a-z]): ([a-z]+)')
```

Let's test our pattern against one of the samples!

```python
>>> pattern.match('1-3 a: abcde')
<re.Match object; span=(0, 12), match='1-3 a: abcde'>
```

It looks like it worked! If it didn't match, then it would have returned `None` (`null`). The return value looks a
little weird when we print it out, but that's ok—what we really want is the groups of the pattern. We can extract each
group with the `.group()` method or get all of them with `.groups()`. Let's get all of them and take a look.

```python
>>> match = pattern.match('1-3 a: abcde')
>>> match.groups()
('1', '3', 'a', 'abcde')
```

Perfect! This has extracted each element we care about into separate elements of a tuple. The only problem left is that
each element is still a string (`str`), but we want the first two elements to be numbers (`int`s). Let's unpack the
tuple, parse the strings into numbers, and re-pack it into a tuple again.

> This could be a situation where we could use a class with named fields. However, since this is just a simple exercise
> (and the tuple will only be used a few times) we'll just stick with an unnamed tuple.

```python
>>> count_min, count_max, letter, password = match.groups()
>>> int(count_min), int(count_max), letter, password
(1, 3, 'a', 'abcde')
```

You may look at the output here and think that we didn't change anything. However, if you look closely, the first two
elements in the tuple are no longer surrounded by single quotes (`'`). In the first example, they were strings
(`str`s). In this example, they are `int`s. Keeping track of data types is very important when programming. Let's
combine all of these pieces into a single `parse_line` function.

```python
LINE_RE = re.compile('(\d+)-(\d+) ([a-z]): ([a-z]+)')
def parse_line(line):
    count_min, count_max, letter, password = LINE_RE.match(line).groups()
    return int(count_min), int(count_max), letter, password
```

Note that I put the pattern into global `LINE_RE` variable. Compiling a regular expression is a non-trivial operation,
and you should avoid doing that in a loop whenever possible. Since we need the same regular expression pattern for every
line, it's better to compile it once and use it many times.

Now that we can parse each line into a tuple with the information we need, we need to determine if the password is valid
or not. Let's review the rules for valid passwords.

> The password policy indicates the lowest and highest number of times a given letter must appear for the password to be
> valid.

Our first goal is to determine how many times the given letter appears in the password. Fortunately for us, Python
includes a `.count()` method on strings (and lists). In our example password, the given letter was `a` and the password
was `'abcde'`. This password contains one `a`, so the return value should be `1`.

```python
>>> 'abcde'.count('a')
1
```

Great! Now we need to check that it is between the minimum and maximum numbers. This is actually a great opportunity to
take advantage of Python's [chained comparisons](https://docs.python.org/3/reference/expressions.html#comparisons). We
want to make sure that the minimum value is less than or equal to the count, which is less than or equal to the maximum
value. Here's what that looks like in Python.

```python
>>> occurrences = 'abcde'.count('a')
>>> 1 <= occurrences <= 3
True
```

With that, we have what we need to write an `is_valid_password` function! Let's put the pieces together.

```python
def is_valid_password(count_min, count_max, letter, password):
    occurrences = password.count(letter)
    return count_min <= occurrences <= count_max
```

Let's test our function against the three test inputs.

```python
>>> lines = ['1-3 a: abcde', '1-3 b: cdefg', '2-9 c: ccccccccc']
>>> tuples = [parse_line(line) for line in lines]
>>> for tup in tuples:
>>>     is_valid = is_valid_password(*tup)
>>>     print(tup, is_valid)
(1, 3, 'a', 'abcde') True
(1, 3, 'b', 'cdefg') False
(2, 9, 'c', 'ccccccccc') True
```

What just happened here? Let me break down a few pieces. First, I created a list of the input lines for the
examples. Then, I used a [list
comprehension](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions) to transform each line into a
tuple using the previously created `parse_line` function. Then, I used a `for` loop to iterate through the list of
tuples. The strange `*tup` syntax is what the Python documentation refers to as [unpacking arguments
lists](https://docs.python.org/3/tutorial/controlflow.html#unpacking-argument-lists). I've heard it referred to most
often as the "splat" operator. It simply takes a tuple (or list or whatever) and uses each element as a different
argument in the function you're calling. So, instead of calling `is_valid_password` with one argument (a tuple), we call
it with four arguments (each element of the tuple). I used it here because I want the `is_valid_password` function to
have reasonable parameter names, but I needed to return a tuple from the `parse_line` function so it could return more
than one value.

Oh, by the way, our code worked! The first and third passwords were valid but the second was invalid (it did not contain
any `b`s but needed at least one).

The last piece is to take a list of passwords and determine how many are valid. For this, we can actually use our
friendly `count` method (and list comprehensions) once again! If we call the `is_valid_password` function on each
password, it will return `True` or `False`. If we then count the number of `True`s in the resulting list, we'll know how
many were valid. Let's try that with our examples and see if it works.

```python
>>> lines = ['1-3 a: abcde', '1-3 b: cdefg', '2-9 c: ccccccccc']
>>> tuples = [parse_line(line) for line in lines]
>>> bools = [is_valid_password(*tup) for tup in tuples]
>>> bools.count(True)
2
```

Looks like it's working. By the way, I don't recommend naming your variables after their data types as a general
practice. I'm using it here to help the reader keep track of data types, especially since Python doesn't have any
explicit data types.

Now it's time to piece everything together! And this time lets use some better variable names.

```python
def num_valid_passwords(input_file):
    lines = get_lines_from_file(input_file)
    password_infos = [parse_line(line) for line in lines]
    password_valids = [is_valid_password(*password_info) for password_info in password_infos]
    return password_valids.count(True)
```

This is a pretty common pattern I follow when writing code. Break pieces into bite-size chunks, then combine them
together with descriptive names in between.

And now for the moment of truth:

```python
>>> num_valid_passwords('input.txt')
483
```

If I plug that into [Advent of Code Day 2](adventofcode.com/2020/) it says I'm correct! Remember, however, that everyone
has different inputs, so your answer will be different.


## Part 2

We have new password validation rules for part two of this exercise. Now, the numbers don't correspond to the minimum
and maximum number of times a letter can appear, but, instead, correspond to indices in the password string. For the
password to be valid, the given letter must appear at exactly one of those indices—but not both! Interesting.

One thing to note is that almost the entire exercise is unchanged—reading the file, parsing the lines, etc. The only
function we need to change is the `is_valid_password` function! What good fortune.

Let's start by getting the letters at the given indices. I'm going to use the third example.

```text
>>> line = '2-9 c: ccccccccc'
>>> index1, index2, letter, password = parse_line(line)
>>> password[index1]
'c'
>>> password[index2]
---------------------------------------------------------------------------
IndexError                                Traceback (most recent call last)
<ipython-input-39-7cfc02a6d36a> in <module>
----> 1 password[index2]

IndexError: string index out of range
```

What happened? Oh, right, this example uses one-based indices, but Python uses zero-based indices. Let's adjust.

```text
>>> line = '2-9 c: ccccccccc'
>>> index1, index2, letter, password = parse_line(line)
>>> password[index1 - 1]
'c'
>>> password[index2 - 1]
'c'
```

Looks like it's working! Although it's hard to tell with a password of all `c`s. Let's try the second example.

```text
>>> line = '1-3 b: cdefg'
>>> index1, index2, letter, password = parse_line(line)
>>> password[index1 - 1]
'c'
>>> password[index2 - 1]
'e'
```

Letters `c` and `e` are indeed at positions 1 and 3 (when using 1-based indexing). Great!

Now we need to determine if **exactly one** of these letters matches the given letter. This is an operation known as
"[exclusive or](https://en.wikipedia.org/wiki/Exclusive_or)". It is true when one _or_ the other is true, but _not_
both. It is used extensively in computer hardware and low-level programming but is seen less often in high-level
programming. However, there is an [exclusive or
operator](https://docs.python.org/3/reference/expressions.html#binary-bitwise-operations), or "XOR" operator, in
Python. Ostensibly, it can only be used on integers and performs a bitwise xor. However, Python lets us use it on
`bool`s as well, and it does exactly what we'd expect it to.

If you're still a little confused, here's how it works with every combination of two `bool`s.

```python
>>> True ^ True
False
>>> True ^ False
True
>>> False ^ True
True
>>> False ^ False
False
```

There are ways we could do this without the XOR operator, but I think it's fun, so let's try it. First, we'll get the
letters at each index (with one-based indices). Then, we'll see if each one matches the given letter. Then, we'll see if
exactly **one** matches.

```python
def is_valid_password(index1, index2, letter, password):
    letter1 = password[index1 - 1]
    letter2 = password[index2 - 1]
    letter1_matches = letter1 == letter
    letter2_matches = letter2 == letter
    return letter1_matches ^ letter2_matches
```

We could reduce this down to a much shorter function if we wanted to, but then we wouldn't have descriptive names and
    would have to remember operator precedence (oh no!). For this post, we'll err on the side of readability. Anyways, let's
try it on our example inputs.

```python
>>> lines = ['1-3 a: abcde', '1-3 b: cdefg', '2-9 c: ccccccccc']
>>> tuples = [parse_line(line) for line in lines]
>>> for tup in tuples:
>>>     is_valid = is_valid_password(*tup)
>>>     print(tup, is_valid)
(1, 3, 'a', 'abcde') True
(1, 3, 'b', 'cdefg') False
(2, 9, 'c', 'ccccccccc') False
```

These results are different from part 1. Now, only the first example is valid. However, that is what we were expecting!
I'll quote the Advent of Code description for an explanation.

```text
- 1-3 a: abcde is valid: position 1 contains a and position 3 does not.
- 1-3 b: cdefg is invalid: neither position 1 nor position 3 contains b.
- 2-9 c: ccccccccc is invalid: both position 2 and position 9 contain c.
```

I think we're ready to try this on our real input! Fortunately, every other function remains exactly the same. Now that
we've changed our `is_valid_password` function, we can just call `num_valid_passwords` again.

```python
>>> num_valid_passwords('input.txt')
482
```

Wow, that was really close to the last answer (`483`). But, as it turns out, this was correct (for my input). Hooray!


## Conclusion

I hope you found this insightful, educational, or entertaining (or all three). I'll see you next time for day 3!



## Improving our solution

There's not much we can do to make our solution significantly faster. However, there are a few things we could do to
make our solution more resilient in case of exceptional circumstances. Here are a few things I can think of off the top
of my head that we didn't worry about but could check for:

  1. What if the file doesn't exist?
  2. What if the line doesn't match the format we expect?
  3. What if the numbers aren't actually numbers? E.g. `1-z a: abcde`.
  4. What if the numbers are negative?
  5. What if the numbers are greater than the length of the password?

These are the kinds of questions you should ask when trying to write reliable code. In fact, in some languages, we would
not have been able to write our code without considering what would happen in most of these situations.

Handling all of these scenarios would make our code much more complex and distract from what I'm trying to
illustrate. So, in these blog posts, I won't be worrying too much about these or similar scenarios. However, these are
important considerations for any code that needs to be more reliable.
