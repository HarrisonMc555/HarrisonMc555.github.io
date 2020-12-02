---
layout: post
title: "Day 0: Getting Set Up"
date: 2020-11-30 00:00:00 -0700
categories: coding adventofcode python
---

## What is Advent of Code?

[Advent of Code](https://adventofcode.com/) is a series of 25 two-part puzzles created each year by [Eric
Wastl](http://was.tl/). They start off fairly simple but eventually require fairly advanced programs to solve.

I decided that, this year, I would document my adventures solving the puzzles in Advent of Code. Feel free to follow
along to see how someone else approaches each problem!

As I explain each step I'll assume you have some background in programming. However, I'll try to make it as simple as
possible so if you're new to programming you can still follow along.


## Getting Set Up

Since this post isn't about a particular day of Advent of Code, I'll instead cover the tools I plan on using throughout
my journey.


### Git

I use [Git](https://git-scm.com/) for all of my personal coding projects. Git is a version control system that is
primarily used to keep track of changes and enhance collaboration between software developers. Even though I don't
collaborate with anyone while working on Advent of Code, I still use Git for the history it provides for my
coding. Plus, I store my solutions on [GitHub](github.com), which uses Git.

You can install Git [here](https://git-scm.com/downloads) or by using your favorite package manager.

If you want to see my code Advent of Code, it is hosted [here](https://github.com/HarrisonMc555/adventofcode). Keep in
mind that not all of my code is a showcase of perfect coding technique, though!


### Python

[Python](https://www.python.org/) is a programming language that is easy to learn and can help with rapid
prototyping. Since each exercise in Advent of Code only needs to solve each problem once, I'm less concerned than I
normally would be with writing code that is performant or maintainable. The fact that I can use Python's
[REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) to rapidly explore various options is
invaluable when exploring various strategies for each exercise.

You can install Python [here](https://www.python.org/downloads/) or by using your favorite package manager.


### IPython

[IPython](https://ipython.org/) is an interactive Python shell that provides syntax highlighting, tab completion, and
history. It is my preferred way to interactively explore programs in Python.

You can install IPython by running the following command, assuming you have already installed Python and `pip` (the
Python package manager):

```bash
pip install ipython
```

## Conclusion

And that's about it! If you're curious, I use [Emacs](https://www.gnu.org/software/emacs/) as my primary text editor for
both Python and blog posts and I use [Jekyll](https://jekyllrb.com/) for the blog itself.
