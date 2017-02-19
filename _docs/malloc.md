---
layout: doc
title: "Malloc"
submissions:
- title: Part 1
  due_date: 02/17/2017 11:59 pm
  graded_files:
  - alloc.c
- title: Part 2
  due_date: 03/06/2017 11:59pm
  graded_files:
  - alloc.c
learning_objectives:
  - Building Character
  - Rite of Passage
  - Memory Allocation
wikibook:
  - "Memory, Part 1: Heap Memory Introduction"
  - "Memory, Part 2: Implementing a Memory Allocator"
---

## Introduction

In the past, you have been using `malloc()` to allocate memory on the heap. In this MP, you will be implementing your own version of `malloc()`. So by the end of this MP, you would theoretically be able to use your own malloc to compile and run any C code.

## Overview

You should write your implementations of `calloc()`, `malloc()`, `realloc()`, and `free()` in `alloc.c`. `alloc.c` will be the only file we test.

Don't modify `mreplace.c`, `mcontest.c`, `alloc-contest.c`. Those files create the environment that replaces the standard glibc malloc with your malloc. These files will be used for testing.

Your `malloc()` must allocate heap memory using `sbrk()`. You may not use files, pipes, system shared memory, `mmap()`, a chunk of pre-defined stack memory, other external memory libraries found on the Internet, or any of the various other external sources of memory that exist on modern operating systems.

## A Bad Example

Memory allocation seems like a mystery, but in actuality, we are making a wrapper around the system call [`sbrk()`](http://linux.die.net/man/2/sbrk). Here's a really simple implementation of `malloc()`:

```
void *malloc(size_t size) {
    return sbrk(size);
}
```

As you can see, when we request `size` bytes of memory, we call `sbrk(size)` to increase the heap by `size` bytes. Then, we return a pointer to this memory, and we're done. Simple!

Here is our implementation of `free()`:

```
void free(void *ptr) {
}
```

This is a "correct" way to implement `free()`. However, the obvious drawback with our implementation is that we can't reuse memory after we are done with it. Also, we have yet to implement `realloc()` and `calloc()`. Finally, we also have to check for errors when we call [`sbrk()`](http://linux.die.net/man/2/sbrk).

Despite all of this, this is still a "working" implementation of `malloc()`. So the job of `malloc()` is not really to allocate memory, it is to keep track of the memory we've allocated so that we can reuse it. You will use methods that you've learned in class and practiced in the mini-valgrind lab to do this.

## Testing Your Code

In order to test your solution against the testers, run `./mcontest` with the tester you want. You MUST do this or your code will be run with the glibc implementation!

Example:

```
./mcontest testers_exe/tester-1
Memory failed to allocate!
[mcontest]: STATUS: FAILED=(256)
[mcontest]: MAX: 0
[mcontest]: AVG: 0.000000
[mcontest]: TIME: 0.000000
```

We've also distributed a bash script `run_all_mcontest.sh` to run all testers. We design the script so that you can
easily test your malloc implementation. Here is how you use the script 

```
./run_all_mcontest.sh
```

It will automatically make clean and make again. And then run each test cases in _testers_ folder. If you want to skip
some test cases, you can do

```
./run_all_mcontest.sh -s 1 2 3 
```

where 1, 2, and 3 are the tests you want to skip. You can skip as many as you like.

Here are what each of our error codes mean:

```
11: Seg Fault
15: Executed Too Long
139: Seg Fault
256, 512, 768, 1024: various errors in tester-# files
16640: Dynamic linking error
17408: Exceeded Memory Limit (2684354560)
```

### Debugging
`./mcontest` runs an optimized version of your code, so you won't be able to
debug with `gdb.` `./mreplace` uses a version of your malloc which is compiled
without optimization, so you can debug with `gdb.` Here's an example, running
tester2 with gdb:

```
gdb --args ./mreplace testers_exe/tester-2
```

### Real programs
Both `mcontest` and `mreplace` can be used to launch "real" programs (not just the testers). For example:

```
# ignore the warning about an invalid terminal, if you get it
./mreplace /usr/bin/less alloc.c
```

or

```
./mcontest /bin/ls
```

There are some programs that might not work correctly under your malloc, for a variety of reasons. You might be able to avoid this problem if you make all your
global variables static. If you encounter a program for which this fix doesn't work, post on piazza!

## Contest

View the malloc contest [here!](http://cs241grader.web.engr.illinois.edu/malloc/)

The malloc contest pits your implementations of memory allocating functions against your fellow students. There are a few things to know:

* The test cases provided will be used for grading. We may also use some real linux utilities (like ls).
* The memory limit is 2.500GB.
* To submit your program into the contest, you simply commit to subversion. Your most recent SVN submission will be fetched somewhat frequently.
* We will assign a score to each of the three categories (max heap, average heap, and total time) based on how well your program performs memory management relative to a standard solution.
* You can pick a nickname in `nickname.txt`. You will show up as this name on the contest webpage.
* On the webpage, each test will have either be green, which signifies that you passed the test, or red, which signifies that you failed the test. Clicking on the failed test will give you more details on the error output of the test.

### Scores and ranking 

Your score will be computed by the following formula:

$$ 100\% \times \frac{1}{3n} \sum_{i=1}^n ((log_b(\frac{time_{reference, i}}{time_{student, i}} + (b-1)) + (log_b(\frac{avg_{reference, i}}{avg_{student, i}} + (b-1)) +(log_b(\frac{max_{reference, i}}{max_{student, i}} + (b-1))) $$

Where:

* n is the number of tests.
* b is the base of log function. Which is 2. 
* _reference_ in the subscript means reference implementation and _student_ means student's implementation.
* $time_{reference, i}$ is the time reference implementation spends on test i. 
* $time_{student, i}$ is the time student spends on test i.
* $avg_{reference, i}$ is the average memory used by reference implementation on test i.
* $avg_{student, i}$ is the average memory used by student implementation on test i.
* $max_{reference, i}$ is the max memory used by the reference implementation on test i.
* $max_{student, i}$ is the max memory used by the student implementation on test i.


__Example 1.__

If a student implementation _x_ performs like the reference implementation, which means it spends the same time and memory like the reference, the score of x will be:

$$
\begin{aligned}
score_x 
&= 
100\%\times \frac{1}{3n} \sum_{i=1}^n ((log_b(\frac{time_{reference, i}}{time_{x, i}} + (b-1)) + (log_b(\frac{avg_{reference, i}}{avg_{x, i}} + (b-1)) +(log_b(\frac{max_{reference, i}}{max_{x, i}} + (b-1))) \\
&= 
100\%\times \frac{1}{3n} \sum_{i=1}^n ((log_b( 1+ (b-1)) + (log_b(1 + (b-1)) +(log_b(1+ (b-1))) \\
&= 100\%\times \frac{1}{3n} \sum_{i=1}^n 3\\
&=  100\%
\end{aligned}
$$


__Example 1.__

If a student implementation _x_ performs three times better than the reference implementation, which means $time_{x, i} = \frac{1}{2}\times time_{reference, i}$ , $avg_{x, i} = \frac{1}{2}\times avg_{reference, i}$, and  $max_{x, i} = \frac{1}{2}\times max_{reference, i}$, x's score will be:

$$
\begin{aligned}
score_x 
&= 
100\%\times \frac{1}{3n} \sum_{i=1}^n ((log_b(\frac{time_{reference, i}}{time_{x, i}} + (b-1)) + (log_b(\frac{avg_{reference, i}}{avg_{x, i}} + (b-1)) +(log_b(\frac{max_{reference, i}}{max_{x, i}} + (b-1))) \\
&= 
100\%\times \frac{1}{3n} \sum_{i=1}^n ((log_b( 3 + (b-1)) + (log_b(3 + (b-1)) +(log_b(3+ (b-1))) \\
&= 100\%\times \frac{1}{3n} \sum_{i=1}^n ((log_b( b+2 ) + (log_b(b+2) +(log_b(b+2)) \\
&= 100\%\times \frac{1}{3n} \times n\times 3log_b(b+2) \\
&= 200\%
\end{aligned}
$$

**WARNING:** Especially as the deadline approaches, the contest page will refresh slower and slower. There are 400 students, 11 test cases, and up to 30 seconds per test case. It will only retest a student's code if it has been updated, but many more students will be updating their code causing longer waits. Start early, and don't become reliant on the contest page by testing locally!

## Grading

Here is the grading breakdown:

* Correctness (75%)
  * Part 1 (25%): tests 1-6 complete successfully - due 02/27 11:59pm
  * Part 2 (50%): tests 1-11 complete successfully - due 03/06 11:59pm
* Performance (25%): Points only awarded if all part 2 testers complete
  successfully - due with part2

There are 11 testcases in total. For part 1 you will be graded using tests 1
through 6. For part 2 you will be graded using tests 1 to 11 (tests 1 through 6
get graded twice).

There are also performance points, which you are only eligible for if you pass
all the testcases. Your malloc will be compared against the `glibc` version of
malloc, and given a performance score as a percentage. For example, if your
malloc is 2 times slower than the `glibc` version of malloc, we will say it
runs in `200%` of the performance of `glibc` malloc. Performance points are
awarded in buckets:

- Better than or equal to 200% of `glibc`: Full 25% awarded.
- 200-300% (exclude 200%, include 300%): 20% awarded.
- 300-400%: 15% awarded.
- 400-500%: 10% awarded.
- 500% and worse: 0% awarded.

So lets work out some scenarios:

* Scenario 1: A student gets tests 1 through 6 working for part1 and misses 2
  tests on part2. Then they get all of the correctness points for part1, 9/11
  of the correctness points for part2 and none of the performance points. Thus
  this student will receive a `(6 / 6) * 25 + (9 / 11) * 50 + 0 = 65.90%`.
* Scenario 2: A student gets none of the tests working for part1 and gets
  everything working for part2 and beats `glibc`. Then they get none of the
  correctness points for part1, 11/11 of the correctness points for part2, and
  the performance points. This student will receive a
  `(0 / 6) * 25 + (11 / 11) * 50 + 25 = 75.00%`.
* Scenario 3: A student gets tests 1 through 6 working for part1, then they get
  all the tests expect test 4 working for part2. Then they get all of the
  correctness points for part1, 10/11 of the correctness points for part2, but
  they will not receive any of the performance points. This student will
  receive a `(6 / 6) * 25 + (10 / 11) * 50 + 0 = 70.45%`.
* Scenario 4: A student gets tests 1 through 6 working for part1, then they get
  all of the tests working for part2, but they never can only get to `350%` of
  `glibc`. In this case, they get all of the correctness points for part 1, all
  of the correctness points for part 2, but only 15% performance points. So,
  they get `(6 / 6) * 25 + (11 / 11) * 50 + 15 = 90`

* We modified the allocation numbers slightly when we actually grade. 