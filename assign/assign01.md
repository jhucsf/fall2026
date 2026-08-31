---
layout: mathjax
title: "Assignment 1: Making Cents"
---

Milestone 1: due Wednesday, September 9th

Milestone 2: due Wednesday, September 16th

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

# Representing Money

When we cover [floating point numbers](..//lectures/lecture05-public.pdf),
we will see that they are unsuitable for representing money because there
are certain fractional values that can't be represented exactly.

The `Money` class is designed to allow exact representations of currency amounts
in currencies where $$1/100$$ of the nominal currency unit is the smallest denomination
that needs to be represented. The basis of this representation is very simply:
a `Money` instance maintains an integer count of how many $$1/100$$ values there are.
In US currency terms, this amounts to counting how many cents (i.e., pennies) there
are in a currency amount. You should use the `uint64_t` data type to represent
the number of one-hundredths of a unit. This means that `Money` can represent
magnitudes from $$0.00$$ up to $$184,467,440,737,095,516.15$$.

## Your Tasks

You have the following general tasks to complete:

1. implement the member functions of `Money`
2. implement unit tests so that all member functions of
   `Money` are tested thoroughly

The member functions have very detailed documentation comments in
`money.h`, which should be precise enough to serve as a specification
of the behavior of each member function.

Although the quality and comprehensiveness of your unit tests is only
a part of the official grading criteria for Milestone 2, you should still
be writing additional unit tests in Milestone 1 so that all of the
member functions required to be implemented in that milestone are tested
thoroughly.

Note that to receive credit for the Assignment, within one week
of submitting Milestone 2, you must meet with the instructor or a CA
for a brief code walkthrough, in which we will ask you (and your
partner if you are working in a pair) to explain your implementation
in some detail.

## Restrictions

You must adhere to the following restrictions.

**Only standard library classes/functions can be used.** You aren't
allowed to use any external libraries in your implementation.
However, you are free (and encouraged) to use any functionality in the
C++ (or C) standard library.

**Only 64-bit and smaller data types can be used.** You aren't allowed
to use any data type whose representation is larger than 64 bits.

**Original code only.** It should go without saying that all of the code
you submit must be your original work. Copying code from an external
source or generating it using AI would be a violation of academic ethics.

## Recommendations and Hints

This section has further recommendations and hints, in no particular order.

### Do Not Use Floating-Point Values

You will not need to use floating point (`float` or `double`) operations.
If you have a problem that you think requires floating point, there is
definitely a way to solve the problem without floating point.
(Talk to the course staff, or ask a question on Courselore!)

### Helper functions

You may implement helper functions as needed to simplify the implementation
of the required public member functions.

### Checking For Overflow

The standard way to determine if an unsigned integer additional overflowed is
the idiom

```c
sum = a + b;
if ( sum < a ) {
  // overflow occurred
}
```

This technique should be useful in implementing the overloaded addition
operator (`operator+`).

Determining whether an unsigned integer multiplication will overflow is a
bit trickier. One good approach is check one of the two factors to find
the largest value it can be multiplied by without overflowing. If the other
factor exceeds this value, then multiplying the factors will overflow.
Let's say we want to compute the product $$a \times b$$, where $$a$$ and $$b$$
are unsigned 64-bit integers. We can compute the value

$$y = \lfloor x/a\rfloor$$

where $$x$$ is the maximum 64 bit unsigned integer value, `UINT64_MAX`.
If $$b>y$$, then the product $$a \times b$$ cannot
be computed without overflow. This technique should be useful in implementing
the overloaded multiplication operator (`operator*`) and the conversion from
`std::string` to `Money` (`from_str()`).

## Subtraction

Because it needs to handle both positive and negative addends and
sums, your `operator+` implementation can be used to implement subtraction.
The idea is that

$$a - b = a + -b$$

So, if you've implemented `operator+` and the unary `operator-`, it should
be trivial to implement the two-operand form of `operator-`.

### Writing Tests

Your unit tests should test each required member function thoroughly.
We recommend that you implement your tests mostly by adding additional
test functions to `money_tests.cpp`, rather than adding new tests
to the provided test functions.

Your tests should try to create "interesting" scenarios for each tested
member function. This includes things like

* zero vs. non-zero values
* negative vs. non-negative values (and combinations thereof)
* smaller vs. larger values
* etc.

A good mindset for testing is that you are an adversary of your own
code, i.e., you are trying to make it break.

## Submitting

Start by exporting a zipfile from your work. Change directory into the
root of your CSF project repository. Run the `ls` command: you should
see a directory called `csf_assign01`.

Run the following command:

```text
git archive -o csf_a1.zip HEAD:csf_assign01
```

This will create a zipfile called `csf_a1.zip` in the root of your
CSF project repository.  Upload this zipfile to Gradescope as
**Assignment 1 MS1** or **Assignment 1 MS2**, depending on which
milestone you would like to submit.

Once you've uploaded the zipfile to Gradescope, you can delete it:

```text
rm -f csf_a1.zip
```
