---
---

# Lecture Notes: Correctness

## Lecture 1

### Module overview
Welcome to the final module! So far, you have spent a great deal of time
learning about concrete tools to help you build software. In this module, we
will look first at some philosophical tools---ways of thinking---and then one
sample concrete tool, [UTest](https://github.com/sheredom/utest.h).

This module will be discussion-based.

### What does it mean for software to be correct?
Possible meanings for software correctness (as discussed in class):

* It does what I want it to
* It follows the specification
* It doesn't crash
* I wrote a proof about it
* The PR/diff/CL was reviewed and accepted by my coworkers
* ...
* It doesn't have any bugs

Some [interesting reading](https://tildesites.bowdoin.edu/~allen/courses/cs260/readings/ch12.pdf)
that we won't assign.

### What is a bug?
Wikipedia has a fine definition.

> A software bug is an error, flaw or fault in a computer program or system
> that causes it to produce an incorrect or unexpected result, or to behave in
> unintended ways.

The interesting parts of this definition are "unexpected results" or
"unintended behaviors". These hint at, but do not explicitly call out, the
existence of a specification for the behavior of a given program. This
specification need not be a three-hundred page tome handed down from management
for a software engineering team to implement. It may be less formal, like a
homework assignment, or even just a vague notion in your mind of what you want
your program to do. If your program deviates from this specification, it has a
bug.

You might ask why software correctness matters at all. In what will later be
described as a crime against Nature, we taught rocks to think and forced them
to run our programs. All those rocks can do are move and compute numbers. So
who really cares if a program doesn't precisely conform to its specification
and the numbers are wrong? Well...

### Why are bugs bad?
A cop-out answer is that the professor of your computer science course has told
you that buggy code will cause you to lose points. For a couple years of your
life, this will suffice.

More broadly, though, software is supposed to help people, and it can’t help
people if it doesn’t perform the way it’s supposed to. Billions of people rely
on software every day for everything from critical infrastructure to medical
equipment to silly games. If there's a one in a million chance that your cancer
radiation therapy machine[^therac25] has a bug that kills you---well, you might
care. Or if the facial recognition software the state employs has a bug that
lands you in a prison cell, you might care.

[^therac25]: The Therac-25 radiation therapy machine is a case study often used
    in engineering ethics courses. Due to a race condition, the machine
    occasionally dosed people with hundreds of times the radiation they should
    have received, injuring several people and killing several people.

On a less serious note, bugs can lead to revenue loss, or wasting people's
time. For personal projects, bugs might be inconsequential, like Bob Ross's
happy little accidents. You won't always be writing code for small projects,
though.

### How do we minimize the number of bugs in software?
Different classes of bugs can be mitigated or outright prevented with different
software practices.

For example, *segmentation faults* and *memory corruption*, which you may have
experienced while writing C++ code, result from a class of bug that is very
unlikely to happen in other programming languages like Python or Java. These
higher-level languages have different *memory models* that outright preclude
these kinds of memory bugs.

Another class of bug, *logic errors*, are easy to introduce when writing
complex code with lots of edge cases like string processing algorithms. Higher
level languages often provide more library functions than lower level ones, and
such functions often provide battle-tested implementations of such algorithms.
Library functions are frequently more correct than a from-scratch
implementation because they have been written and revised by many people.

Other logic errors are preventable by employing *mathematical proofs*. Tools
like [Coq](https://coq.inria.fr/) and [Isabelle](https://isabelle.in.tum.de/)
make it possible for programmers to write proofs about properties of the
systems they are building and have them automatically checked. Coq can then
generate a program for you that has been proven correct[^specification-errors].

[^specification-errors]: Formal verification is sometimes cited as a way to
    eliminate bugs altogether. After all, if your program has been
    mathematically proven to be correct, and if we have defined "correct" to
    mean "no bugs," it by definition cannot have bugs. But there's a flaw in
    this reasoning: proofs like those generated by Coq and Isabelle only prove
    that a program confirms to a specification that you---the person using
    them---provide. For example, if you write a program that returns the number
    4, and you tell Coq that your program ought to return 4, there's a pretty
    good chance it can prove it "correct." But if this specification is itself
    inaccurate---the customer wanted the program to return the number 2, for
    example---or incomplete, the proof is worthless.

When proving a program correct is impossible or intractable, it's almost always
possible to fall back on *testing*. Tests manually exercise your code with some
inputs and check the results against a set of known-correct answers. A good
test suite on a software project is often a mark of high attention to detail
and a reasonable proxy for correctness. Tests also have one advantage over
proofs: since they exercise the code in a real environment, they validate the
environment as well as the code. For example, if you accidentally rely on
undefined behavior from your code and then upgrade your compiler to one that
produces a different result, your test suite will let you know. As Donald Knuth
once said, "Beware of bugs in the above code; I have only proved it correct,
not tried it."

Lastly, software development practices can help. For a multi-person software
project, having a required code review step in the development process can help
catch bugs and otherwise raise the bar. People reviewing code may notice edge
cases that the original author did not think of, request that the author write
tests for those edge cases, and improve the quality of the proposed code
change.

In this module, we're going to focus primarily on writing tests as a means for
ensuring software correctness. Tests are not the only way to make your software
more correct, but they are the easiest to immediately apply and reason about.

### "Best practices"
While we intend for everything we teach to be helpful, our advice won't always
apply in every situation. Use your best judgement. Read
[this tweet](https://twitter.com/garybernhardt/status/1433474928024735748) by
Gary Bernhardt.

## Lecture 2

Even simple software has edge cases. In CS 15, for example, one assignment
involves writing a `delete` function to remove an element from a binary search
tree and maintain the BST invariant. This function alone had several cases: the
node is `NULL`; the node has no children; the node has one child; the node has
two children. Even though you probably should write unit tests, that's a
manageable number of cases to test manually.

Complex software has many more edge cases. Business requirements often have to
take into account the Real World, which is much messier than a binary search
tree. In one day, you might have to think about software performance, adding a
new feature, complying with internal guidelines, complying with a new law
passed by the state, and complying with a new law passed in a different
country. There are so many cases to consider. In isolation, they are tricky
problems to keep correct. When combined, the cross product can swiftly become
completely and utterly unmanageable to keep correct.

The good news is that there *are* reasonably well-established software
practices to untangle this huge mess of code. Writing tests is helpful, yes,
but there are some other auxiliary practices that can help make your tests even
more effective.

### Automated tests

If you, the programmer, have to run tests manually after every change you make,
you will probably not run them very often. And, worse, if the test suite takes
a long time to run, you may run them even less. Automating this process is
instrumental to both ensuring that the tests run and reducing cognitive load.

If you think back to our source control module, where we introduced a Pull
Request-based collaborative workflow, there is a great place to insert an
automated test run: run tests on each commit in a pull request. Collaborative
version control providers often expose an API surface for building and running
tests. Having a green checkmark per change is a good signal to you, the
programmer, and your colleagues, that you have not broken anything, and also
pass your new tests.

### Invariant of the green main branch

There's a bit of an implicit assumption that makes automated tests useful:
expecting the main branch to be passing tests ("green"). If you maintain the
invariant that the main branch always passes tests, it's straightforward to
tell if your changes break anything.

If tests fail on your change, then it is most likely that the change introduced
a bug---a *regression*. It's also possible that the change exposed a bug that
already existed elsewhere and was not tested, in which case the signal is still
useful to the person submitting the change.

If the tests are failing or flaky on the main branch, though, the signal is
much less useful to anyone submitting a change to the project. It's very
difficult to use the test failure to a particular root cause.

If you want to get really nitpicky about this, having tests run with every
change *still* is not enough. Consider a case where multiple engineers are
writing landing changes concurrently. Generally, tests run for each change:

```
change A        (pass)
change B        (pass)
```

But this does not account for concurrent landing of A and B. If A and B both
land at the same time, there will be a "land race" and they will land in some
non deterministic order. It may happen that A and B conflict with one another
and one change causes the other to fail to build or fail to pass tests.

```
change A    *before*    change B        (fail)
change B    *before*    change A        (build fail)
```

This is a common reason that projects also enable "land-time" tests, ensuring
that the linearization of concurrent landing commits still passes tests.
Land-time tests build and run the project before every commit to the main
branch.

### The real world and pebbles in a stream

As much as some people might wish, you are not writing code in a spherical
vacuum fixed in time. Even if you manage to write perfect bug-free code, which
is extraordinarily unlikely, your code will at some point interact with other
software and with hardware. This code is also extraordinarily unlikely to be
bug-free.

You will have bugs. Even if they aren’t bugs in the state-of-the-world at time
of writing code, the world changes. It’s not about how good you are as a
programmer. Your code interacts with the toolchain of the programming language
in which it's written: the compiler or the runtime. It interacts with the
operating system. With the filesystem. With the network. Your code may run well
when compiled with Clang but crash when compiled with GCC, and the reason may
be that your code implicitly took advantage of a subtle clause in the C
standard that allows the two compilers to implement a behavior differently.

The third-party library you use might also have bugs. Or if it does not have
bugs, something changed with the last version. Perhaps the email software *du
jour* has an upgrade that changes a timeout and causes delivery
failures[^email]. Or perhaps part of the code literally changes behavior over
time, because it relies on the current date to do something.

[^email]: This is a fun, if apocryphal, story underscoring the point.
    https://web.mit.edu/jemorris/humor/500-miles

In all of these cases, writing tests, frequently running tests, and ensuring
that the test environment is the same as the "production" environment (whatever
that may be), will ease your pain.

Google’s new CPU failures [paper](https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s01-hochschild.pdf)

Facebook hardware failure
[paper 1](https://research.fb.com/wp-content/uploads/2020/03/Optimizing-Interrupt-Handling-Performance-for-Memory-Failures-in-Large-Scale-Data-Centers.pdf)
and [paper 2](https://arxiv.org/pdf/2102.11245.pdf)

## Lecture 3

In this lecture, we will explore a method for writing useful tests for
software.

### Where to start

Start with the specification: what should the function do? Test what is
specified. Imagine a function `isEven` that must return `true` if the number
given was even, and `false` otherwise:

```c
bool isEven(int num);
```

In order to pick good test cases for this function, we should consider the
cases mentioned in the specification: an even number; an odd number. Let's
write some tests.

```c
TEST(MySoftwareModule, IsEvenWithOddNumberReturnsFalse) {
  EXPECT_EQ(isEven(7), false);
}

TEST(MySoftwareModule, IsEvenWithEvenNumberReturnsTrue) {
  EXPECT_EQ(isEven(8), true);
}
```

These tests exercise a small sample of the very large space of even and odd
numbers (about two billion each) that could be passed into this function. The
tests assume a "regular" implementation, as opposed to a "silly"
implementation. For example, these tests would not do a very good job checking
the correctness of the following implementation of `isEven`:

```c
bool isEven(int num) {
  switch (num) {
    case 0: case 2: case 4: case 6: case 8: return true;
    case 1: case 3: case 5: case 7: case 9: return false;
    // TODO: add the rest of the numbers
    default: return false;
  }
}
```

This is a somewhat reductive example, but such "stub" functions are not
uncommon in production codebases. The band-aid solution to this is generally to
make the unimplemented case abort the program, and have a unit test checking
for an abort when passing in unimplemented input.

Sometimes your programming environment might change how you think about unit
testing your code. In the above C environment, there are no exceptions to
handle and types are fixed at compile time. There is no need to test that you
will always get a `bool` back from `isEven`---it is guaranteed. In general,
there is no need to test infrastructure guarantees. In other programming
environments, you might have fewer things guaranteed by the compiler or
runtime. Let us consider a Python language equivalent of `isEven`:

```python
def is_even(num):
  if num in (0, 2, 4, 6, 8): return True
  if num in (1, 3, 5, 7, 9): return False
  # TODO: add the rest of the numbers
  return False
```

There is one big difference in Python compared to C: you will notice that there
are no types to be found! This is because Python is a dynamically typed
language. It does not require you to annotate your variables or function
signatures with types. This means that it's possible to pass other types such
as a string, or a floating point number, or something else entirely to
`is_even`, and it won't be a compile-time error. It won't be a run-time error
either, in this case; `is_even` will return `False`.

So what *should* it do, given a non-integer? That's to be written in the
specification, and tested in the unit tests. In this case, appropriate tests
might ensure that the code raises an exception, or that it returns some
sentinel value.

### Other things to test

When testing a function, there are two main approaches: blackbox testing, and
whitebox testing.

When writing blackbox unit tests, assume the function definition is hidden. All
you get is the signature. What inputs can you think of to break the function?
Let's look at `isEven` again.

```c
bool isEven(int num);
```

Some interesting values come to mind: what happens if you pass zero? A negative
number? A very large number? Integers in C are bound to some platform-dependent
size. What happens if you pass `INT_MIN` or `INT_MAX` to the function? Without
looking at the body of this function in particular, there is not much else to
test in this style. Other signatures may leave room for more interesting test
cases---maybe they operate on strings or floating point numbers or something
more complicated.

When writing whitebox unit tests, on the other hand, assume you have access to
the current function definition. What test cases might break it? One common
strategy is coverage-based testing---ensuring that all code paths through the
function are tested. This requires figuring out values that pass and fail
various conditionals, exercise behaviors of loops, and so on. Some of the
conditionals may be implicit, if they happen in functions called inside the
function you are testing.

<!-- TODO: example of coverage based testing -->

Although coverage is a good way to find code paths you haven't tested, you
should be wary about blindly chasing 100% coverage. There are plenty of cases
where a code path just isn't that interesting or error-prone; writing tests for
such paths purely to improve your coverage metric can waste time and hide the
few tests that actually matter among lots that don't.

### Naming tests

You may notice that the unit tests for the `isEven` function have very verbose
names. This is not because the course staff are enthusiastic Java
programmers---we are not---but instead because the names serve as
documentation.  Imagine writing a test case with the name `IsEvenTests`, that
expects different results with a bunch of different numbers. With your current
implementation of `isEven` and with your current context, that may be fine. But
you will likely come back to these tests in a year or three, with the
implementation of `isEven` having completely changed, and wonder what made
these numbers so special. Or perhaps your coworker will wonder about these
things, because they did not write the code in the first place.

Having descriptive test names also separates concerns for the testing harness.
If only part of the `isEven` function breaks, it would be nice to know at a
glance *which* part broke. `IsEvenWithOddNumberReturnsFalse` failed? Well. Now
you know what to take a look at.

### Which functions to test

Write tests for code you write. Sometimes when writing tests for a function *f*
that calls another function *g*, it is tempting to write tests that directly or
indirectly check the output of *g*. Assuming that *g* is independently well
tested (by you or by its author), this is unnecessary[^mocking].

[^mocking]: Some people take extra care to avoid this by using a technique
    called *dependency injection* and *mocking* the third-party functions, but
    the community is divided on its merits. We will talk more about this later.

Sometimes the software you are building is meant to be consumed by other
programmers, either in the form of a library, or a service. Either way, your
software will provide an Application Programming Interface---an API. Because
this is the primary way your users will be interacting with your software, it
is imperative these API functions be well-tested. You will likely also write
smaller building blocks---internal functions, classes, or other services. Test
these, too.

### Maxims

**Test small units of code as directly as possible.** Ideally, your functions
should be small and have simple lives. And ideally tests should call the
functions they are testing directly. That's the simplest way to make sure that
you're actually calling the intended function, that you're actually calling
with the variant you intended to test, and that something else did not affect
your results.

**Avoid "round trips" through layers of software.** Round trips, including
nested function calls, or network requests, or disk I/O, or other things,
increase the amount of noise in your testing.

**Avoid stateful computation.** If your test requires some setup, such as
creating a file on the disk, or adding a table to a database, you will likely
run into issues with concurrency. Tests that have shared mutable state may
break when run concurrently, leading to a slow, sequential test suite.
Additionally, this violates the "round trips" maxim above.

**It’s not a test unless you watch it fail.** Make sure your tests are running!
If they have never failed, it's entirely possible that you are testing the
wrong function, or not running your tests, or something somewhere is very, very
broken.

## Lecture 4

The function we wrote above is fairly straightforward to test. If you wanted
to, it is not difficult to enumerate the entire space of inputs for `isEven`
and check their results. On a modern computer, this would take no more than a
second. So what makes software difficult to test? And, implicitly, should we
factor software to be easier to test?

In the previous lecture, we alluded to parts of this difficulty with the
testing maxims: test small units of code; avoid round trips; avoid state. Let's
break those down.

### Test small units of code

With any luck, you will have been advised to write code with single-purpose,
easily-understandable, composable units. While there is always some discourse
about how big functions should be and where to divide them, it's easier to test
smaller chunks than bigger chunks. Consider the case of coverage-based testing
from before: the more conditionals you have to reason about, the harder your
test cases get. If your function has multiple conditional early returns, some
large `if` statements, and a loop, you're in for a rough time.

Consider also the case where you have a "compound" function that does two
things. Maybe `void setAgeAndHeight(int age, int height)`. It's not clear how
to test this function---do you write one test that tests two of its behaviors?
Write two tests, each testing one behavior? What about the complex behavior
space for valid and invalid inputs? It would be easier to test if it were split
into two functions that could be tested independently: `void setAge(int age)`
and `void setHeight(int height)`. Then it is reasonably straightforward to
write:

```c
TEST(PeopleSoft, SetAgeSetsAge) {
  Person person;
  person.setAge(50);
  EXPECT_EQ(person.age(), 50);
}

TEST(PeopleSoft, SetHeightSetsHeight) {
  Person person;
  person.setHeight(70);
  EXPECT_EQ(person.height(), 70);
}
```

And, as before, if you have a test failure in the future, the failing test
should clearly point to the function that broke.

### Avoid round trips

Often it is tempting to test the internals of a bit of code indirectly by
calling it via another function. Maybe this is because the top-level function
requires fewer parameters, or less state setup, or neatly packages up its
results---whatever the reason may be, try to resist this temptation. If at all
possible, call the function directly by name.

As an example, we can look at the unfortunate `setAgeAndHeight` function from
earlier. Pretend it is implemented as follows:

```c
void setAgeAndHeight(int age, int height) {
  setAge(age);
  setHeight(height);
}
```

Instead of testing `setAge` by calling `setAgeAndHeight` and then reading the
age in the test, call `setAge` directly! This can be tricky when writing
private methods. There are several ways around this, of which we recommend two.
The first is to make the method public. This is not always an option; some
methods should stay private. The second, which is not always available either,
is to use the language-specific feature that exposes private methods to tests.
C++, for example, has `friend` declarations.

We don't mean to discourage you from writing *integration tests*---tests that
run a whole suite of software as one unit to ensure that it works
together---but instead encourage you to keep your unit tests unit-y.

* Avoid state
* I/O
* Randomness
* Filesystem / network

`void computeAndPrintResult()`

That function does not return anything and only produces its output as an I/O
side effect. It would be easier to test if it were split into two functions
`int computeResult()` and `void printResult(int)`.

(API surface is I/O heavy and not mockable; software is fundamentally
nondeterministic; etc)

Factor software for testability. Test from within, and optionally from without.

### Parting thoughts

When you change your software, do you run the tests of everybody who uses your
software?

There is a programming language called Rust and software tooling for Rust
programmers to publish package their software into units called *crates*. With
every release of the Rust compiler, the Rust team runs the compiler on many
crates to check for regressions[^crater].

[^crater]: https://github.com/rust-lang/crater

Software engineers who write programming language infrastructure at large
companies (Clang at Google and Facebook, Go at Google, HHVM at Facebook, etc)
are not doing so in a vacuum; most of the time, they have an internal customer
that uses their compiler or language runtime. These teams tend to also test
their releases against their customers' tests.

What do you do if you find breaking behavior surfaced by a large integration
test written by your client?
