---
layout: mathjax
title: "Assignment 4: Parallel Quicksort"
---

**Due**: Monday, November 16th by 11 pm

Assignment type: **Pair**, you may work with one partner

# Parallel Quicksort

In this assignment, you will

1. Complete a partially-implemented program which uses the
   [Quicksort](https://en.wikipedia.org/wiki/Quicksort) sorting algorithm
   to sort a file containing a sequence of 64-bit signed integers;
   specifically, add code to use the [mmap](https://man7.org/linux/man-pages/man2/mmap.2.html)
   system call to create a shared mapping of the data in the file so that
   the contents of the file appear as an array in memory
2. Use child processes to execute the recursive calls in parallel, in order
   to take advantage of multiple CPU cores to complete sorting more
   quickly

## Grading Criteria

Your assignment grade will be determined as follows:

* Handling command line arguments, using open and mmap to map the file: 20%
* Parallel sorting using subprocesses: 50%
* Experiments and report: 15%
* Error reporting: 5%
* Design and coding style: 10%

## Getting Started

Download [csf\_assign04.zip](csf_assign04.zip) and unzip it. You will be
modifying the file `parsort.c`.

## Quicksort

The starter code has a correct implementation of quicksort, so you don't really
need to completely understand it in order to do the assignment. However, quicksort
is a very simple and elegant algorithm, so it's not too hard to explain.

The basic idea of quicksort is based on *partitioning* a sequence into
"left" and "right" partitions. Partitioning works as follows:

1. An arbitrary "pivot" element in the sequence is chosen
2. The sequence is re-arranged so that all of the elements less than
   the pivot value occur before the pivot value (this is the
   "left" partition), and all the elements
   greater than or equal to the pivot value occur after the pivot
   value (this is the "right" partition)

Once a sequence is partitioned, it can be sorted by recursively sorting
the left partition and the right partition.

Let's say that we want to sort all elements of a sequence called *arr*
between the indices *start* (inclusive lower bound) and *end*
(exclusive upper bound.) A pseudo-code implementation of quicksort
might look something like this:

```
quicksort( arr, start, end ) {
  n = end - start
  if ( n < 2 )
    // base case: sequence is trivially already sorted
    return

  // the partition function returns the index where
  // the pivot element ended up
  mid = partition( arr, start, end )

  quicksort( arr, start, mid )
  quicksort( arr, mid + 1, end )
}
```

Paritioning a sequence with $$n$$ elements is $$O(n)$$. Assuming that
quicksort generally chooses a pivot element such that the left and right
partitions are roughly equal in size, the overall running time of
quicksort is $$O(n \log n)$$ in the average case.

### Parallel quicksort

Because the two recursive calls to quicksort operate on completely independent
parts of the array, there is no reason why they can't execute in parallel
on different CPU cores. This approach to converting a sequential algorithm
to allow for parallelism is known as [fork/join](https://en.wikipedia.org/wiki/Fork%E2%80%93join_model),
and in general can be used to speed up the execution of divide and
conquer algorithms.

Eventually we will learn about [threads](https://en.wikipedia.org/wiki/Thread_(computing))
as a mechanism for executing instructions on multiple CPU cores. For this assignment,
we will use [child processes](https://en.wikipedia.org/wiki/Child_process) to
execute the recursive calls in parallel. Normally, a data structure in a parent
process, such as an array being sorted, would not be shared with a child process,
since by default, child processes do not share memory with their parent process.
However, if a shared file mapping is created, the memory containing the data
of the mapped file is shared between parent and child processes, which means
a child process can participate in sorting the data in the file.

## Tasks

To complete the assignment, you will need to do the following:

1. Fill in missing details in the `main` function to open the file
   containing the data to sort, determine its size, and use the
   [mmap](https://man7.org/linux/man-pages/man3/qsort.3.html) system call
   to create a shared mapping of the file's contents
2. Verify that the program can correctly do sequential sorting
   (without any parallelism)
3. Modify the program so that when the number of elements to sort is
   greater than the specified *parallel threshold*, the recursive calls
   to the `quicksort` function are executed by child processes
4. Do experiments to measure the observed speedup when sorting a
   large file with varying parallel threshold values, and write a report
   explaining the observed running times

Note that when the number of elements to sort is less than or equal
to the parallel threshold, the elements are sorted using a call
to the [qsort](https://www.man7.org/linux/man-pages/man3/qsort.3.html)
function, which will sort the elements efficiently using a single
CPU core.

### Task 1: open file, determine its size, map its data

You will start by modifying the program to open the file, determine its
size, and using `mmap` to create a shared memory mapping of its contents.

First, you will need to use the [open](https://man7.org/linux/man-pages/man2/open.2.html)
syscall to open the file in read-write mode and get a file descriptor:

```c
int fd = open(filename, O_RDWR);
if (fd < 0) {
  // file couldn't be opened: handle error and exit
}
```

Next, `mmap` will need to know how many bytes of data the file has. This can be
accomplished using the [fstat](https://man7.org/linux/man-pages/man3/fstat.3p.html) system
call:

```c
struct stat statbuf;
int rc = fstat( fd, &statbuf );
if ( rc != 0 ) {
    // handle fstat error and exit
}
// statbuf.st_size indicates the number of bytes in the file
```

Note that in addition to finding the size of the file, you should also
compute the number of `int64_t` elements in the file's data.
(I.e., divide the size in bytes by `sizeof(int64_t)`.) 

Once the program knows the size of the file, creating a shared read-write mapping will
allow the program, and all its descendants, to modify the file in-place in memory:

```c
int64_t *arr;
arr = mmap( NULL, file_size_in_bytes, PROT_READ | PROT_WRITE,
            MAP_SHARED, fd, 0 );
close( fd ); // file can be closed now
if ( arr == MAP_FAILED ) {
    // handle mmap error and exit
}
// *arr now behaves like a standard array of int64_t.
// Be careful though! Going off the end of the array will
// silently extend the file, which can rapidly lead to
// disk space depletion!
```

Passing in `NULL` for the requested mapping address gives `mmap` complete freedom to
choose any address in memory as the base address for the mapping. Since we don't care
where the file's data ends up in memory, so long as we can access it, this is what we want.
Similarly, we want to map the entire file, so we set the offset to zero.

Note that before the `main` function returns, it should call
[munmap](https://man7.org/linux/man-pages/man2/mmap.2.html) to unmap
the file contents from memory. This should look something like

```c
munmap( arr, file_size_in_bytes );
```

### Task 2: verify that sequential sorting works

Once the program can map the file's data, it should work correctly to implement
sequential sorting of the data in the file. Two helper programs are provided
to help you verify whether sorting is working:

* `gen_rand_data` creates a file containing random bytes (the `parsort` program
  will interpret its contents as an array of `int64_t` values)
* `seqsort` is a purely sequential C++ sorting program that can serve as
  an "oracle" to make sure your `parsort` program produced the correct results

Here's an example of using these two programs to test `parsort` on a one
megabyte ($$2^{20}$$ bytes) randomly generated data file:

```text
$ make gen_rand_data parsort seqsort
gcc -g -Wall -c gen_rand_data.c -o gen_rand_data.o
gcc -o gen_rand_data gen_rand_data.o
gcc -g -Wall -c parsort.c -o parsort.o
gcc -o parsort parsort.o
g++ -g -Wall -std=c++17 -c seqsort.cpp -o seqsort.o
g++ -o seqsort seqsort.o
$ ./gen_rand_data 1M test_data_1.bin
Wrote 1048576 bytes to 'test_data_1.bin'
$ ./parsort test_data_1.bin 65536
$ ./gen_rand_data 1M test_data_2.bin
Wrote 1048576 bytes to 'test_data_2.bin'
$ ./seqsort test_data_2.bin 
$ diff test_data_1.bin test_data_2.bin 
$ echo $?
0
```

If the `diff` command does not produce any output, and exits with exit code 0
(as indicated by the `echo $?` command), it means that `parsort` and `seqsort`
both produced the result data, indicating that `parsort` worked correctly.
Note that the parallel threshold value passed to `parsort` was 65536 in the
example above, but it should work for any threshold value.
Also note that the data created by the `gen_rand_data` program is "pseudo-random",
meaning that it is deterministic, and in general will always generate the same
sequence of bytes.

### Task 3: use child processes to execute the recursive calls

To allow the recursive sorting to make use of multiple CPU cores
(when the number of elements being sorted is greater than the
parallel threshold), you will use the `fork()` system call to create
one child process for each recursive call.

Using the [fork](https://man7.org/linux/man-pages/man2/fork.2.html) system
call is straightforward:

```c
pid_t child_pid = fork();
if ( child_pid == 0 ) {
  // executing in the child
  // ...do work...
  if ( /* work was done successfully */ )
    exit( 0 );
  else
    exit( 1 );
} else if ( child_pid < 0 ) {
  // fork failed
  // ...handle error...
} else {
  // in parent
}
```

The code labeled `...do work...` above would be the computation you
want the child to perform, i.e., recursively sorting part of the array.

Eventually, the parent should wait for the child to complete.
This can be done using the [waitpid](https://man7.org/linux/man-pages/man3/wait.3p.html)
system call:

```c
int rc, wstatus;
rc = waitpid( child_pid, &wstatus, 0 );
if ( rc < 0 ) {
  // waitpid failed
  // ...handle error...
} else {
  // check status of child
  if ( !WIFEXITED( wstatus ) ) {
    // child did not exit normally (e.g., it was terminated by a signal)
    // ...handle child failure...
  } else if ( WEXITSTATUS( wstatus ) != 0 ) {
    // child exited with a non-zero exit code
    // ...handle child failure...
  } else {
    // child exited with exit code zero (it was successful)
  }
}
```

Note that in the above code snippets for `fork` and `waitpid`, there are a number
of situations where a failure can occur:

1. the child can't be created (i.e., `fork()` fails)
2. the parent doesn't successfully learn the result of the child
   (i.e., `waitpid()` fails)
2. the child isn't successful (it doesn't exit normally, or it
   exits with a non-zero exit code)

If any of these situations occurs, the parent process needs to make sure that
it returns 0 from the `quicksort` function to indicating that sorting failed.

One detail that can be tricky to get right is to make sure that for any
child process that is created successfully, the parent *definitely* attempts
to wait for the child (by calling `waitpid()`.) Because of the numerous
ways in which errors can occur, and because there will in general be *two*
child processes (one for each recursive call), the control paths can get
quite complex. A good way to reduce this complexity is to introduce a data
type to represent a child process. An instance of this child process
represents:

1. whether the child was created successfully, and if so, its process id
   (`pid_t` value)
2. what state the child process is in: is it running, or if we waited
   for it to complete, was the wait successful, and if so, what is the
   process's exit code

Let's say you have a `Child` data type representing a child process.
Your code to do parallel sorting might look like this:

```c
Child left, right;
left = quicksort_subproc( arr, start, mid, par_threshold );
right = quicksort_subproc( arr, mid + 1, end, par_threshold );

quicksort_wait( &left );
quicksort_wait( &right );

left_success = quicksort_check_success( &left );
right_success = quicksort_check_success( &right );
```

In this code, the `quicksort_subproc` function is responsible for creating
a child process to do recursive sorting. The `quicksort_wait` function waits
for a child process to complete. The `quicksort_check_success` function
checks whether a child process completed successfully. You are welcome
to use this approach, although you're not required to.
In general, you will need to guarantee

1. if the parent creates a child, it is guaranteed to wait for the child
   to complete, and
2. if any errors occur, they are correctly reported by having `quicksort`
   return 0 rather than 1

### Task 4: experiments and analysis

To make sure that your `parsort` program is exhibiting the expected degree of
parallelism, we would like you to perform an experiment where you create
a random data file of 16 megabytes in size, and then time sorting this file
multiple times, while adjusting the threshold to achieve increasing amounts
of parallel execution.

You should run the following commands:

```
make clean
make
mkdir -p /tmp/$(whoami)
./gen_rand_data 16M /tmp/$(whoami)/data_16M.in
cp /tmp/$(whoami)/data_16M.in /tmp/$(whoami)/test_16M.in
time ./parsort /tmp/$(whoami)/test_16M.in 2097152 
cp /tmp/$(whoami)/data_16M.in /tmp/$(whoami)/test_16M.in
time ./parsort /tmp/$(whoami)/test_16M.in 1048576
cp /tmp/$(whoami)/data_16M.in /tmp/$(whoami)/test_16M.in
time ./parsort /tmp/$(whoami)/test_16M.in 524288
cp /tmp/$(whoami)/data_16M.in /tmp/$(whoami)/test_16M.in
time ./parsort /tmp/$(whoami)/test_16M.in 262144
cp /tmp/$(whoami)/data_16M.in /tmp/$(whoami)/test_16M.in
time ./parsort /tmp/$(whoami)/test_16M.in 131072
cp /tmp/$(whoami)/data_16M.in /tmp/$(whoami)/test_16M.in
time ./parsort /tmp/$(whoami)/test_16M.in 65536
cp /tmp/$(whoami)/data_16M.in /tmp/$(whoami)/test_16M.in
time ./parsort /tmp/$(whoami)/test_16M.in 32768
cp /tmp/$(whoami)/data_16M.in /tmp/$(whoami)/test_16M.in
time ./parsort /tmp/$(whoami)/test_16M.in 16384
rm -rf /tmp/$(whoami)
```

The `parsort` commands start with a completely sequential sort,
and then tests with increasing degrees of parallelism.
For example, at the smallest threshold of 16384 elements, there
will be 128 processes doing sequential sorting at the
base cases of the recursion.

We have provided a shell script called `run_experiments.sh` which
runs these commands, so to collect your experimental data you could
just run the command

```
./run_experiments.sh
```

We suggest using one of the numbered ugrad machines (`ugrad1.cs.jhu.edu`
to `ugrad12.cs.jhu.edu`) to do your experiment. When you log in, you can
run the `top` command to see what processes are running. (Note that you
can type `q` to exit from `top`.) If any processes
are consuming significant CPU time, you should consider logging into
a different system, until you find one where no processes are consuming
significant CPU time.

When you run the commands, copy the output of the `time` command. The
`real` time will indicate the amount of time that elapsed between when
the program started and exited, which is a pretty good measure of
how long the sorting took.  You *should* see that decreasing the
threshold decreased the total time, although depending on the number
of CPU cores available, eventually a point of dimimishing returns
will be reached.

In your `README.txt`, write a brief report which

1. Indicates the amount of time that your `parsort` program took
   so sort the test data for each threshold value, and
2. States a reasonable explanation for *why* you saw the times you did

For \#2, think about how the computation unfolds, and in particular,
what parts of the computation are being executed in different processes,
and thus which parts of the computation could be scheduled by the
OS kernel in parallel on different CPU cores. We don't expect a completely
rigorous and in-depth explanation, but we *would* like you to
give an intuitive explanation for the results that you observed.

### Note on the autograder

Passing the autograder will be a necessary but insufficient condition for full credit.
This means that you may still lose functionality points, even if you pass all of the
autograder tests. Due to the nature of testing parallel programs, there  will be a
significant number of points up for manual review, so please structure your code
accordingly. Some of the things we may manually verify (bot not limited to) are:

* Ensuring that your implementation is actually parallel. (Your
  [experiments](#experiments-and-analysis) should have already allowed you to
  determine whether your program is exhibiting any parallel speedup.)
* Ensuring that sorting is actually accomplished.
* Ensuring that you did not leave zombies around during execution.
* Ensuring that a "reasonable" number of children are created for a given threshold
  and data size value.

## Submitting

Edit the `README.txt` file to include the report and summarize each team member's contributions.

Create a zipfile of your solution using the command

```
make solution.zip
```

Submit your zipfile to Gradescope as **Assignment 4**.
