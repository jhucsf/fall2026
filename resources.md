---
layout: default
title: "Resources"
category: "resources"
---

This page has links to useful resources.

# Information

This section has links to some information resources you might find useful.

## Practice problems and exams

Review materials for Exam 1:

* [Exam 1 practice questions](resources/exam1review.html), [Solutions](resources/exam1review-solutions.html)
* [Midterm, Spring 2020](resources/midterm-spring2020.pdf) (Questions 1–3), [Solution](resources/midterm-spring2020-soln.pdf)
* [Exam 1, Fall 2021](resources/exam01-fall2021.pdf), [Solution](resources/exam01-fall2021-soln.pdf)
* [Exam 1, Fall 2025](resources/exam01-fall2025.pdf), [Solution](resources/exam01-fall2025-soln.pdf)

Review materials for Exam 2:

* [Exam 2 practice questions](resources/exam2review.html), [Solutions](resources/exam2review-solutions.html)
* [Midterm, Spring 2020](resources/midterm-spring2020.pdf) (Question 4), [Solution](resources/midterm-spring2020-soln.pdf)
* [Final exam, Spring 2020](resources/final-spring2020.pdf) (Questions 1–3), [Solution](resources/final-spring2020-soln.pdf)
* [Exam 2, Fall 2021](resources/exam02-fall2021.pdf), [Solution](resources/exam02-fall2021-soln.pdf)
* [Exam 2, Fall 2025](resources/exam02-fall2025.pdf), [Solution](resources/exam02-fall2025-soln.pdf)

Review materials for Exam 3:

* [Exam 3 practice questions](resources/exam3review.html), [Solutions](resources/exam3review-solutions.html)
* [Final exam, Fall 2019](resources/final-fall2019.pdf) (Questions 4–5), [Solution](resources/final-fall2019-soln.pdf)
* [Final exam, Spring 2020](resources/final-spring2020.pdf) (Questions 4–5), [Solution](resources/final-spring2020-soln.pdf)
* [Exam 3, Fall 2021](resources/exam03-fall2021.pdf), [Solution](resources/exam03-fall2021-soln.pdf)
* [Exam 3, Fall 2025](resources/exam03-fall2025.pdf), [Solution](resources/exam03-fall2025-soln.pdf)

## General resources

* [Floating Point Visually Explained](https://fabiensanglard.net/floating_point_visually_explained/)

## x86-64 assembly language exercises

* [Assembly language mini-exercises](resources/assemblyMini.html)
* [Assembly language exercise](resources/assembly.html), [solution](resources/asmExerciseSoln.zip)
* [Assembly language exercise 2 (more challenging)](resources/assembly2.html)

## x86-64 assembly programming resources

* [CSF Assembly Language Tips & Tricks](https://jhucsf.github.io/csfdocs/assembly-tips-v0.1.2.pdf)
  * This is a very comprehensive guide to x86-64 assembly language written by
    Max Hahn, focusing on issues that are important for CSF
* [Brown x64 cheat sheet](https://cs.brown.edu/courses/cs033/docs/guides/x64_cheatsheet.pdf)
* [Brown gdb cheat sheet](https://cs.brown.edu/courses/cs033/docs/guides/gdb.pdf)
* [CMU summary of gdb commands for x86-64](http://csapp.cs.cmu.edu/3e/docs/gdbnotes-x86-64.pdf)

## Style Guidelines
* The [style guidelines](resources/style.html) state our coding style expectations.

# Software

This section covers the software you'll be using in working on programming assignments.

## Linux

For the programming assignments, you will need to use a recent x86-64 (64 bit) version of Linux.

**Important**: the code you submit is required to run correctly on Ubuntu 22.04, since
that is the version of Linux that we use in [Gradescope](https://www.gradescope.com/) autograders.

Here are some options for getting your development environment set up.

You can use the [CS ugrad machines](https://support.cs.jhu.edu/wiki/Linux_Clients_on_the_CS_Undergrad_Net)
to do your development work. The ugrad machines use a recent version of
[Fedora Linux](https://fedoraproject.org/). Correctly-written
code will work the same way on Fedora and Ubuntu.

You can install [Ubuntu 22.04](https://releases.ubuntu.com/22.04/) directly on your
computer.  This is a good option if you are comfortable installing operating systems
from installation media.

On Windows 10 and Windows 11, you can use the
[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
(WSL).  Once WSL is enabled, you can install Ubuntu 22.04 from the Microsoft Store.  Make sure that
you install the [tools](#tools) listed below.  Using WSL is an excellent option if you are
comfortable doing your development work inside a terminal session.

Note that if you are using an Apple Silicon-based (ARM) Mac computer,
then (as far as we know) there aren't any good
options for setting up a local development environment.  Virtualization won't work
because the computer doesn't use an x86-64 CPU. However, using
[Visual Studio Code](https://code.visualstudio.com/) connected to an SSH
workspace which accesses your ugrad account is a good option if you
want to use an IDE. If you are having issues using VS Code to connect
to your ugrad account, the CS IT team has some troubleshooting advice:

<https://support.cs.jhu.edu/wiki/Trouble_Logging_In_Using_VS_Code_With_Remote_SSH_Plugin>

## Valgrind

Code that relies on undefined behavior (such as expecting the value of an uninitialized variable to
be 0) could very well behave differently on the autograder than it does when you run it on your development
system. Make sure that you test your code using [valgrind](https://valgrind.org/) to make sure there
are no memory errors.

## Tools

Some of the tools you'll want to have are:

* gcc
* g++
* make
* ruby
* valgrind
* git

All of these are available by default on the Ugrad computers.

To install on an Ubuntu-based system:

```
sudo apt-get install gcc g++ make ruby valgrind git
```

You'll also want to install a text editor.  [Emacs](https://www.gnu.org/software/emacs/) and [Vim](https://www.vim.org/) are good options:

```
sudo apt-get install emacs vim
```

## Vim setup
[Tutorial](resources/vim) on setting up Vim

## Using Git
[Github ssh authentication](resources/github-ssh.html): How to use ssh to access
your private repositories on Github
