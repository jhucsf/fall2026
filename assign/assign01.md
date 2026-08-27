---
layout: mathjax
title: "Assignment 1: Making Cents"
---

Milestone 1: due Wednesday, September 9th

Milestone 2: due Wednesday, September 16th

<div class='admonition caution'>
  <div class='title'>Important</div>
  <div class='content' markdown='1'>
This is a preliminary assignment description. Details might change,
and it's possible that the eventual Assignment 1 might not be anything
like what you see here.
  </div>
</div>

# Overview

In this assignment, you will implement a C++ class called `Money`
which represents an amount of money. Specifically, an instance of
`Money` represents the total number of one-hundredths of a
currency unit. For example, if a `Money` object represents an amount
of US currency, then it is counting cents, since a US cent is equal
to one-hundredth of the currency unit, the US dollar. Most
(but not all) world currencies, including the US dollar and the
Euro, use 1/100 of a unit as the smallest denomination, although there
are exceptions.

## Milestones, Grading Criteria

Milestone 1 (15% of the assignment grade):

* Implementation of functions (15%)
  - constructors
  - destructor
  - assignment operator
  - `get_whole()`
  - `get_frac()`
  - `is_negative()`

<div class='admonition danger'>
  <div class='title'>Important!</div>
  <div class='content' markdown='1'>
Milestone 1 is intended as a warm-up, since you might not have
written C++ code in a while. For that reason, it is a very
lightweight milestone. Milestone 2 will require significantly
more work.
  </div>
</div>

Milestone 2 (85% of the assignment grade):

* Implementation of functions (65%)
  - `operator+`
  - `operator-`
  - `operator*`
  - `operator/`
  - `operator-` (unary minus)
  - comparison operators (`<`, `>`, etc.)
  - `to_str()`
  - `from_str()`
* Quality and comprehensiveness of your unit tests (10%)
* Design and coding style, code walkthrough (10%)

## Getting Started

If you already have access to your CSF project repository, clone it.

If you don't yet have access to your CSF project repository:

* Get access as soon as you can (contact the course staff, we can help)
* Create a git repository to use temporarily until you get access to
  your CSF project repository

In a terminal, change directory to your local clone of your
CSF project repository.

Download the starter code:

```text
curl -O https://jhucsf.github.io/fall2026/assign/csf_assign01.zip
```

Unzip the zipfile, and add, commit, and push the starter code to your
CSF project repository:

```text
unzip csf_assign01.zip
git add csf_assign01
git commit -m'add assignment 1 starter code to project repo'
git push
```

Now you can delete the starter code zipfile:

```text
rm csf_assign01.zip
```

Change directory into the `csf_assign01` subdirectory within your CSF
project repository:

```text
cd csf_assign01
```

You will be adding code to `money.cpp` to implement the various member functions,
and also adding unit tests to `money_tests.cpp` to test those implementations.

You can compile and run the unit tests as follows:

```
make depend
make -j
./money_tests
```

The last command will run all of the unit tests. If you only want to run one
specific test function. For example, if you only want to run the tests in the
`test_get_whole()` function, you can run the command

```
./money_tests test_get_whole
```

This is very useful when you want to focus on testing one specific member function.
