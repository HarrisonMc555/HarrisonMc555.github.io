---
layout: post
title: "2020 Day 4: Passport Processing"
date: 2020-12-04 12:00:00 -0700
categories: adventofcode
---

## Part 1

This post explores day 4 of Advent of Code 2020! You can find the description for part 1 of this
exercise [here](https://adventofcode.com/2020/day/3).


### The goal

For this exercise, we need to determine which passports are valid. A passport can contain the
following fields:

```text
byr (Birth Year)
iyr (Issue Year)
eyr (Expiration Year)
hgt (Height)
hcl (Hair Color)
ecl (Eye Color)
pid (Passport ID)
cid (Country ID)
```

For a passport to valid, it must contain all of these fields except the `cid` (Country ID) field,
which is optional.


### Parsing the input

In most of the exercises we've seen so far, the input has been in a very structured, consistent
format. However, in this exercise, things can be a little different. Let's look at the example batch
file.

```text
ecl:gry pid:860033327 eyr:2020 hcl:#fffffd
byr:1937 iyr:2017 cid:147 hgt:183cm

iyr:2013 ecl:amb cid:350 eyr:2023 pid:028048884
hcl:#cfa07d byr:1929

hcl:#ae17e1 iyr:2013
eyr:2024
ecl:brn pid:760753108 byr:1931
hgt:179cm

hcl:#cfa07d eyr:2025 pid:166559648
iyr:2011 ecl:brn hgt:59in
```

The passports are separated by blank lines, but each field can be separated by spaces _or_
newlines. This makes things a little more tricky! Let's walk through a few possibilities and think
if they would work. We can't just read each line one at a time, because that would put different
parts of the same passport in different sections. We could maybe go through each line at a time and
add it to a running total until we found a blank line, but that sounds a little tedious and
error-prone. Is there a better solution?

What if we read the entire batch file at once and then split on blank lines? Python strings have a
`.split()` method that can take a string (not just a single character), so we should be able to
split on `\n\n` (two newlines in a row). Let's try it!

For convenience, I'm going to save the example batch file in a file named `example.txt` so we can
work with it. I hope this will make it easier to follow along than if I used my personalized
`input.txt` file.

```python
>>> text = open('example.txt').read()
>>> passports_texts = text.split('\n\n')
>>> passports_texts[0]
'ecl:gry pid:860033327 eyr:2020 hcl:#fffffd\nbyr:1937 iyr:2017 cid:147 hgt:183cm'
```

Looks good! Now we need to get one section for each `key:value` pair. The issue here is that we have
two kinds of [delimiters](https://en.wikipedia.org/wiki/Delimiter): spaces and newlines. The
`.split()` method only accepts a single delimiter, so we could either split on spaces or on
newlines, but not both. Darn.

However, let's look a little closer at the `.split()` method. Python has a built-in help system, so
let's use it.

```text
>>> help(str.split)
Help on method_descriptor:

split(self, /, sep=None, maxsplit=-1)
    Return a list of the words in the string, using sep as the delimiter string.

    sep
      The delimiter according which to split the string.
      None (the default value) means split according to any whitespace,
      and discard empty strings from the result.
    maxsplit
      Maximum number of splits to do.
      -1 (the default value) means no limit.
```

Well would you look at that. If we don't pass in a value for `sep` (the "separator" or delimiter),
then it splits it according to _any_ whitespace. That's exactly what we want!

```python
>>> passport = passports_texts[0].split()
>>> passport
['ecl:gry', 'pid:860033327', 'eyr:2020', 'hcl:#fffffd', 'byr:1937', 'iyr:2017', 'cid:147', 'hgt:183cm']
```

The last step is to break apart the `key` from the `value` in the `key:value` pairs. Looks like
another job for `.split()`! That method has definitely been useful today. This time we'll use `:` as
the delimiter.

```python
>>> [pair.split(':') for pair in passport]
[['ecl', 'gry'], ['pid', '860033327'], ['eyr', '2020'], ['hcl', '#fffffd'], ['byr', '1937'], ['iyr', '2017'], ['cid', '147'], ['hgt', '183cm']]
```

That looks pretty good. My only complaint is that each `key`/`value` pair is in a list, not a
tuple. That seem like a minor nitpick, but lists and tuples imply different things and have
different capabilities. A list can be any size and can be dynamically resized. A tuple has a fixed
size and cannot be resized. In this case, each `key`/`value` pair should always have exactly two
elements: the `key` and the `value`. It should never have three elements! Let's modify our code
slightly to use tuples instead.

```python
>>> [tuple(pair.split(':')) for pair in passport]
[('ecl', 'gry'), ('pid', '860033327'), ('eyr', '2020'), ('hcl', '#fffffd'), ('byr', '1937'), ('iyr', '2017'), ('cid', '147'), ('hgt', '183cm')]
```

Ah, much better! Much better for me, at least. Let's combine each part into a single function! Or
actually several functions.


```python
def parse_passports(text):
    passports_text = text.split('\n\n')
    return [parse_passport(pt) for pt in passports_text]

def parse_passport(passport_text):
    pairs = passport_text.split()
    return [tuple(pair.split(':')) for pair in pairs]
```

And a quick test to see if things worked.

```python
>>> text = open('example.txt').read()
>>> passports = parse_passports(text)
>>> passports[0]
[('ecl', 'gry'), ('pid', '860033327'), ('eyr', '2020'), ('hcl', '#fffffd'), ('byr', '1937'), ('iyr', '2017'), ('cid', '147'), ('hgt', '183cm')]
```

Looks great!


### Validate passports

Now we need to determine which passports are valid. To review:

> For a passport to valid, it must contain all of these fields except the `cid` (Country ID) field,
> which is optional.

We simply need to check and make sure that all required keys are present. That's actually not too
bad! First, let's create a list of all the required keys.

```python
required_keys = ['byr', 'iyr', 'eyr', 'hgt', 'hcl', 'ecl', 'pid', 'cid']
```

Oh, wait, the `cid` isn't required! Let's remove that.

```python
required_keys = ['byr', 'iyr', 'eyr', 'hgt', 'hcl', 'ecl', 'pid']
```

For each passport, what we would like to do is to iterate through the required keys and see if they
are present. However, there's not a super-simple (or super-efficient) way to do that if each
passport is a list of tuples. What if our passports were, instead, a dictionary/map?

A dictionary, or map, is a data structure that maps a collection of keys to their corresponding
values. Huh, that's exactly what each passport is. I guess we should have used a dictionary from the
beginning.

Fortunately, in Python, it's pretty easy to convert a list of `key`/`value` pairs into a
`dict`. Simply use the `dict` function!

```python
>>> dict(passports[0])
{'ecl': 'gry', 'pid': '860033327', 'eyr': '2020', 'hcl': '#fffffd', 'byr': '1937', 'iyr': '2017', 'cid': '147', 'hgt':
'183cm'}
```

Well that was easy. Let's modify our original `parse_passport` function to return a `dict` instead.

```python
def parse_passport(passport_text):
    pairs = passport_text.split()
    return dict(pair.split(':') for pair in pairs)
```

The careful observer will note that I've removed the call to `tuple` and I'm using a list
comprehension...without a list? As it turns out, 1) the `dict` function can take a sequence of
two-element lists, so we don't need to wrap it in a tuple, and 2) Python has [generator
comprehensions](https://www.pythonlikeyoumeanit.com/Module2_EssentialsOfPython/Generators_and_Comprehensions.html#Creating-your-own-generator:-generator-comprehensions),
which remove the need to explicitly wrap it in a list (when I didn't need one anyways). Neither of
these are important details, though, so feel free to ignore them if you're confused.

Let's see how our new function works.

```python
>>> passports = parse_passports(text)
>>> passports[0]
{'ecl': 'gry', 'pid': '860033327', 'eyr': '2020', 'hcl': '#fffffd', 'byr': '1937', 'iyr': '2017', 'cid': '147', 'hgt': '183cm'}
```

Looks good! Now we're finally ready to write our `is_valid_passport` function. For that, we'll
simply go through the each required key and see if it's present in the passport.

```python
def is_valid_passport(required_keys, passport):
    for key in required_keys:
        if key not in passport:
            return False
    return True
```

If you're new to Python, the `not in` syntax may seem weird, but it does what you would expect:
checks if a key is present in a `dict`. If any of the keys are missing, we return `False` because
the passport is missing a required key. If we get to the end of the list of required keys and we
haven't quit yet, then all of the required keys must have been present. So we return `True`. Let's
see if it works for each of our example passports.

```python
>>> for passport in passports:
...     print(is_valid_passport(required_keys, passport))
...
True
False
True
False
```

This matches what the explanation of the example says—the first and third are valid, the second and
fourth are invalid. Looks like we're set to go!


### Counting valid passports

Now we need to get all of our passports and count how many are valid. We'll use the same strategy we
used in [day 2](../02/day-2-password-philosphy.html), namely list comprehension and `.count(True)`.

```python
def count_valid_passports(required_keys, passports):
    valids = [is_valid_passport(required_keys, p) for p in passports]
    return valids.count(True)
```

Let's try that for the example.

```python
>>> count_valid_passports(required_keys, passports)
2
```

Looks good! It looks like all of our pieces are coming together. Time to see if we can get the right
answer for the _real_ input.

```python
>>> text = open('input.txt').read()
>>> passports = parse_passports(text)
>>> count_valid_passports(required_keys, passports)
182
```

And submitted...and yes! That is the correct answer. Hooray!


## Part 2

Things just got a lot more interesting. Instead of just making sure each key _exists_, we now have
to make sure that they actually are valid values. To make things more complex, each key has a
different set of rules!

Let's go through them one at a time.


### Birth Year (`byr`)

The rules are:

```text
four digits; at least 1920 and at most 2002.
```

This is fairly straightforward. It must be an integer between 1920 and 2002.

```python
def is_valid_byr(value):
    num = int(value)
    return 1920 <= num <= 2002
```

Let's run this against the test cases.

```python
>>> is_valid_byr(2002)
True
>>> is_valid_byr(2003)
False
```

Looks good!


### Issue Year (`iyr`)

The rules are:

```text
four digits; at least 2010 and at most 2020.
```

This is almost the exact same thing as birth year. We can copy and paste and change the name and numbers.

```python
def is_valid_iyr(value):
    num = int(value)
    return 2010 <= num <= 2020
```

We don't have any test cases for these. Here are some I made up:

```text
iyr invalid: 2009
iyr valid: 2010
iyr valid: 2020
iyr invalid: 2021
```

Let's run this against these test cases.

```python
>>> is_valid_iyr(2009)
False
>>> is_valid_iyr(2010)
True
>>> is_valid_iyr(2020)
True
>>> is_valid_iyr(2021)
False
```

Looks good!


### Expiration Year (`eyr`)

The rules are:

```text
four digits; at least 2020 and at most 2030.
```

This is once again almost the exact same thing as birth year. Here's the implementation; I'll forego the test cases for this one.

```python
def is_valid_eyr(value):
    num = int(value)
    return 2020 <= num <= 2030
```


### Height (`hgt`)

This one is interesting. It has a number followed by either `cm` or `in` for centimeters or inches
respectively. Each type of unit has different rules.

```text
a number followed by either cm or in:
    - If cm, the number must be at least 150 and at most 193.
    - If in, the number must be at least 59 and at most 76.
```

This requires two things:

  1. Split the word into the number and the suffix (unit).
  2. Use the appropriate test for the given unit.

We're going to use the `.endswith()` method to test for which suffix (if any) the value ends
with. Then, we'll take off the last two letters to get the number. To do that, we'll use a slice
with a negative index. Python lets you use negative indices to index from the end (instead of the
beginning) of a string/list.

```python
def is_valid_hgt(value):
    if value.endswith('cm'):
        num = int(value[:-2])
        return 150 <= num <= 193
    elif value.endswith('in'):
        num = int(value[:-2])
        return 59 <= num <= 76
    else:
        return False
```

If it doesn't end with either `cm` or `in`, then it's invalid. Let's run our method against the
given test cases.

```python
>>> is_valid_hgt('60in')
True
>>> is_valid_hgt('190cm')
True
>>> is_valid_hgt('190in')
False
>>> is_valid_hgt('190')
False
```

Looks good!


### Hair Color (`hcl`)

Here are the rules for hair color:

```text
a # followed by exactly six characters 0-9 or a-f.
```

In the previous section we used the `.endswith()` method. Here we can use the corresponding
`.startswith()` method. Then, we'll use string slices again and use the `all()` method to check if
every character is valid. We can define a helper method to easily check if each character is valid.

One thing we'll take advantage of is the ASCII numeric definition of characters. Since both the ten
Arabic digits and the letters `a` through `f` are listed in order in ASCII, we can check for those
two ranges fairly easily.

```python
def is_valid_hcl(value):
    if not value.startswith('#'):
        return False
    return all(is_valid_hcl_char(c) for c in value[1:])

def is_valid_hcl_char(c):
    is_0_through_9 = '0' <= c <= '9'
    is_a_through_f = 'a' <= c <= 'f'
    return is_0_through_9 or is_a_through_f
```

Let's try these with the given test cases:

```python
>>> is_valid_hcl('#123abc')
True
>>> is_valid_hcl('#123abz')
False
>>> is_valid_hcl('123abc')
False
```

Looks good!


### Eye Color (`ecl`)

Here are the rules:

```text
exactly one of: amb blu brn gry grn hzl oth.
```

This one is fairly simple: it must be in a given set of values. This is the perfect opportunity to
use a `set`! A `set` can tell us if a value is in it in constant time. Using a list would be fine,
but it would have to go through the entire list every single time—using a `set` is much faster.

```python
VALID_EYE_COLORS = set(['amb', 'blu', 'brn', 'gry', 'grn', 'hzl', 'oth'])
def is_valid_ecl(value):
    return value in VALID_EYE_COLORS
```

Let's try that.

```python
>>> is_valid_ecl('brn')
True
>>> is_valid_ecl('wat')
False
```

Looks good!


### Passport ID (`pid`)

Last one! At least, last one that has any kind of validation. Here are the rules:

```text
a nine-digit number, including leading zeroes
```

In the past, we've converted the number to an int and checked for a range of values. Just for kicks
and giggles, let's do this one a little differently.

Let's first check for the length of the string (should be `9`). Then, we'll make sure every
character is a digit.

```python
def is_valid_pid(value):
    if len(value) != 9:
        return False
    return all(c.isdigit() for c in value)
```

Could we have used the `.isdigit()` method for `is_valid_hcl`? Yes, we could have. Good point!
Anyways, let's try those test cases.

```python
>>> is_valid_pid('000000001')
True
>>> is_valid_pid('0123456789')
False
```

Looks good!


### Bringing it all together

Now, we need to call these functions on the corresponding values of each passport. Our first check
will always be "does this value exist?". Our second question will be "is this value valid for this
key?", which will be different for each key.

To reduce the amount of duplicated code we would need to write, we can make a lists of all of the
validator functions with their corresponding keys. Let's do that here.

```python
VALIDATORS_AND_KEYS = [
    (is_valid_byr, 'byr'),
    (is_valid_iyr, 'iyr'),
    (is_valid_eyr, 'eyr'),
    (is_valid_hgt, 'hgt'),
    (is_valid_hcl, 'hcl'),
    (is_valid_ecl, 'ecl'),
    (is_valid_pid, 'pid'),
]
```

Now, we can iterate through each validator function and test them all.

```python
def is_valid_passport(passport):
    for validator, key in VALIDATORS_AND_KEYS:
        if key not in passport:
            return False
        value = passport[key]
        if not validator(value):
            return False
    return True
```

This looks very similar to our last `is_valid_passport` function. The biggest difference is that, in
addition to checking for the _presence_ of each key, we're also making sure the value passes the
corresponding validator function.

Finally, let's modify our `count_valid_passports` function since we're not using the `required_keys`
parameter any more.

```python
def count_valid_passports(passports):
    valids = [is_valid_passport(p) for p in passports]
    return valids.count(True)
```

Let's try it out with the list of invalid passports!

```python
>>> text = open('invalid-passports.txt').read()
>>> passports = parse_passports(text)
>>> count_valid_passports(passports)
0
```

Looks good! How about the valid passports? There should be `4`.

```python
>>> text = open('valid-passports.txt').read()
>>> passports = parse_passports(text)
>>> count_valid_passports(passports)
4
```

Wonderful! And against the real input?

```python
>>> text = open('input.txt').read()
>>> passports = parse_passports(text)
>>> count_valid_passports(passports)
109
```

And that's the right answer!


## Conclusion

This was a long one. I went quickly through each key since there were so many. Hopefully I'll be
able to give more in-depth analysis of future days, assuming they don't have quite as many moving
pieces.
