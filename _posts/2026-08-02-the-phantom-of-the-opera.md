---
title: The Phantom of the Opera
subtitle: Inverting Coverage Metrics to Chase Superfluous Accidental Complexity
seriestitle: "Testing in the Age of LLM Assisted Coding"
tags: [Testing, LLM]
layout: post
toc: true
---

<div class="flowers">
👻
</div>

# The Verification Gap

With LLM assisted coding, the question of verifying the solution has become more
pressing. As popping out lines of code has become much faster, the verification
gap, sometimes called "review gap" or "understanding gap", has been revealed as
the bottleneck of software programming as a business enabling activity.

This changes the way we should think about testing. If we can find a way to
define our tests so that we can specify the expected program in such a tight way
that the LLM agent cannot confabulate bad code, and consider the
actual implementation safe and ready to ship without an extra review, that might
help to relax the bottleneck.

For the sake of argument, a simplified example for modelling the situation
where the output of the LLM is too expensive to review: Imagine we want to
verify the correctness of a function that is supposed to double an integer, but
we don't want to look at its implementation:

```python
def double(x: int) -> int:
    ... # implementation hidden from us
```

We can start the TDD cycle by writing a simple test:

```python
assert double(1) == 2
```

And then hand implementation over to the agent. Can we be sure that this testing
is sufficient without "manually" inspecting the internals of `double`, though?
What if the LLM came up with this code?:

```python
def double(x: int) -> int:
    if x == 1: return 2
    return 0
```

Our test would pass, yet the function would not do what we want it to. The LLM
"cheated on us", it was "cutting corners" and just focused on getting the green
light in our test driven development cycle.

(This example is overly simple for illustrative purposes, but the reasoning can
be expanded to any codebase.)

The problem is: Is there a way to verify correctness of the program without
testing all possible input values?

There is. And the good news is: we can do this without having to inspect the
code manually, using a programmatic and deterministic tool, that luckily already
exists:

# Inverting Coverage Metrics

Besides running the tests, we can also verify that our tests cover all the
execution paths, using existing code coverage tools.

If you use code coverage metrics, people will tell you that the goal is not to
reach 100%.[^Coverage] Some folks say it should be "above 80%", but any number
here is funny nonsense, trying to replace a qualitative judgement with a random
number. The underlying insight is that there is no point in adding tests that do
not increase your confidence of the software's being correct -- and that's
right. But, if you find yourself (or your team) gaming the metrics, you got the
point of the code coverage metrics wrong in the first place, and no number
anyone pulling out of their hat can change that.

[^Coverage]: Some even say [_100% Code Coverage is a
    Lie_](https://www.youtube.com/watch?v=p1xZ-Ni2t8Q), or [_100% Code Coverage
    is Useless_](https://www.youtube.com/watch?v=Zs2IpqHzchw)

If we want to reduce the amount of review work we want to do, we can invert
the use of code coverage metrics: Instead of adding tests that don't increase
our confidence about shipping the code (i.e. gaming the metrics), just to make
the code coverage metrics happy (which would just create more review work), we
should tell the agent to simplify the implementation. If we specify the solution
first, then have the agent build against it, and then send the agent into a
cycle where it adapts the code until 1. all the tests pass and 2. the code
coverage is 100% while 3. the initial tests never changed, then tests work as an
effective harness proving that any flaws in the implementation are already in
the specification.

The different paths of execution partition the space of possible input values
into distinct _equivalence classes_.

# Ghost Paths

If we follow TDD with a human writing the specs, and the agent writing the
implementation, the cases covered in our specs describe the _semantic partition_
of the input space.

If the path partition of the program the agent produces is more complex than the
semantic partition defined in the specs, some of those paths are _ghost paths_:
They are paths that are not justified by the essential complexity encoded in
the specifications.

# Make Illegal States Unrepresentable

That implies that the agent violated another important rule of programming,
expressed in a famous quote by Yaron Minsky:

> Make illegal states unrepresentable.

With ghost paths existing, some divisions in the path equivalence classes are
never actually supposed to make a semantic difference, but look like they are
possible to make a significant difference from the perspective of static code
analysis. In this case, we can tell the agent to go back and cut off the
overshooting accidental complexity of the program -- and, most important for
reducing the review gap, we can do this in an automated loop, without investing
human review capacity.

There are specific techniques we can tell the agent to apply. The two most
important tools for doing this are:

1. Early validation and reject.
2. Defining control flow through declarative types or objects that represent
   legal states.

One great example for tighter control of the legal states is the way Rust
defines the switch statement (called `match` in Rust). Other than like (for
instance) Kotlin's `when` construct or Python's `match`, Rust enforces coverage
of all possible cases. Combined with the inverted code coverage approach, this
will force the agent to write tighter type definitions, to eventually avoid
building a ghost path just to satisfy an overly broad type definition.

We don't need to switch to Rust, but we can enforce this strictness in other
languages by using static code checkers in laxer programming languages.
