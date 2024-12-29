---
title: The Limits of My World
subtitle: Paradigms in Programming
layout: post
tags: Collaboration Culture
---

<div class="flowers">
🦁
</div>

Discussions in programming are not always useful. The internet is full of
opinionated, endless and frustrating discussions about coding questions that
lead --- nowhere.

In discussions, there are two useful skills: One is the skill to say the right
things. When I was younger, less experienced and more insecure, I used to focus
on this one (I still sometimes do, unfortunately). The other skill is way more
important, however: It is the skill to ask the right questions.

Had I understood the idea I would like to talk about in this post, it would have
spared me a lot of confusion and frustration and it would have helped me asking
better questions.

The concept that would have helped me is the concept of _paradigms_.


# What is a Paradigm in Software Development?

One way to understand what a _paradigm_ is, is to see the word as a synonym for
_culture_. I am sure there are a lot of fruitful analogies that can be drawn
between a culture and a software paradigm. But as inspiring as this analogy is,
it does not provide a stringent definition. Here is a better definition:

> A _paradigm_ is a network of related decisions, i.e. decisions that influence
> each other.

Any decision is influenced by some contextual factors, which are in turn shaped
by other decisions that have been made before. The decision at hand will in turn
influence future decisions. This [path
dependence](https://en.wikipedia.org/wiki/Path_dependence){:target="_blank"}{:rel="noopener
noreferrer"} _is_ the paradigm.

Writing this post, I am surprised how simple the idea is, and how I was still
blind for this for a long time. Seeing those endless discussions mentioned above
gives me the impression that there are a lot of people out there who are not
aware of this either. This article is for them (including my past
self).


# A Paradigm is Invisible from the Inside

When we fail to discuss in a constructive way, it is often because we fail to
notice that we are living in different paradigms. This is the advice I would
love to give to my past self:

> Whenever you enter a new domain, be it because you start using a new
> programming language, or even a different library, or be it because you start
> working in a new team or company, you should try to explore the existing
> paradigm, to make your contributions to the discussions more fruitful.

When operating in a new domain, we are thrown into a net of path dependencies
-- decisions that have been made in the past and that can only be changed at a
high price (effectively: never). Often, paradigms are implicit, and hence
unknown to newcomers. Often, the people who work within the paradigm are unaware
of its existence, at least partially. It was only after I had worked in
different companies and with different programming languages (mostly Python,
JavaScript, Elixir, and Ruby) that I understood that some things can be done in
many completely different ways -- and all of them can be considered "clean" or
"correct", or even just feel "right".

There is a systemic reason for this: Inside a paradigm, it is hard to realize
its implicit premises because they are always given. They form what we consider
"normal", they form the limits of our world, and there is no way we can be aware
of them (because it would require seeing our world from outside).

Only when we shift context, we can become aware of the principles that underpin
the paradigm we previously worked in. Changing the paradigm is eye-opening, and
I want to compare it to a culture shock.[^elixir]

[^elixir]: The strongest and most joyful moment of broadening my horizon in that
    way happened to me when I had the pleasure to work with Elixir. Working with
    Elixir, a beautiful functional programming language, taught me a completely
    new perspective on writing software, and I backported a lot of its
    principles into writing python code, focussing more on declarative
    programming instead of wading knee-deep through imperative control
    statements.

When we discuss a single question without understanding the paradigm it is
influenced by, our discussion will be pointless. Two simple examples can
illustrate how failing to understand paradigmatic differences can lead to
sophisticated, but fruitless arguments:


# Examples

## Considering `NULL` as Equivalent to `FALSE`

The smallest paradigm consists of a single convention. If a certain convention
is widely accepted, it makes sense to follow it, as it shapes what others will
expect when they read our code. A trivial example is the use of snake-case in
Python. We _could_ do it differently, but choosing Python as a programming
language locks us into using snake-case, although this decision is obviously
arbitrary.

Another simple example is how NULL values are used. Both in Python and in Ruby,
the NULL type (`None` in Python, `nil` in Ruby) is falsey, i.e. it evaluates to
`False` / `false` under a boolean check. In Ruby, it seems to be a convention
that a method which returns a boolean can simply return `nil`, and this is
considered equivalent to `false`. (At least this was the case in a codebase I
worked on.)

One thing that influences this choice is a Ruby convention: In Ruby, names of
functions returning a boolean conventionally end with a `?`. This is kind of the
most rudimentary typing system in a language that is otherwise untyped. So, when
you call such a function, and get a `nil`, it is simply convention to treat it
like `false`. Sometimes, it spares you writing a separate `return false`:

```ruby
def check?(x)
  if (x > 12)
    return true
  end
  # implicitly returning nil here spares us the effort of writing return false
```

Another influence on the matter is whether you have a codebase with type
annotations.

In Python, especially if you follow the widely adopted approach of using type
annotations for your functions, I would consider it a code smell to use `None`
as a drop in for `False`. The type annotation of the return value of a function
would be `() -> bool | None` instead of a simple `() -> bool`. Looking at the
signature, one would expect that `None` indicates some exceptional scenario.

```python
def dirty_check(x: int) -> bool | None:
    if x > 12:
        return True


def clean_check(x: int) -> bool:
    return x > 12
```

So, in a typed Python codebase, you actually have to write _more_ code to return
`None` instead of `False`, while in Ruby, you get away with writing less code.

And both ways are OK -- if the expectations of the people working on the
codebase are aligned. Coming from a typed Python codebase, the Ruby paradigm was
foreign to me, and my mind attached judgmental words such as "dirty", "hacky",
and "abuse" to it. Now I think that this reflects my lack of orientation and
insecurity I felt in the paradigm that was new to me.

I could make a case that in the "Ruby way",[^ruby] we loose the significance of
`nil` (we cannot use it as an additional signal). I used to believe this was a
somewhat sophisticated argument. Now I think it is actually quite narrow-minded:
There are hundred ways to send a signal of exceptional behavior, and to use
`None` as a return value is just another convention. So when I thought that
returning `nil` as a replacement for `false` would be "conceptually wrong",
maybe that was just a post-rationalization of the fact that I never thought
about doing this in the first place, not because it was a "code smell", but
because it would simply have required more typing instead of less in the Python
paradigm I grew up with.

This illustrates how not understanding the limits of your world leads to worse
discussions. It leads to making some argument to justify your habit, but it does
not help understanding each other.

[^ruby]: I am not sure that what I described above is actually a convention that
    is shared across the whole Ruby community, it was just used in the one
    codebase I worked with.


## Should Tests Live in a Separate Directory?

Another good example for paradigmatic difference is the question whether unit
tests should be written adjacent to the code they are testing.

In Python, Ruby, and JavaScript, it is very common to separate tests from source
code, usually in different folders in the root of the project, so that the test
directory structure reflects the directory structure of the source code. In
Rust, the convention is exactly the opposite: unit tests are supposed to sit
right next to the code they are testing. Is this a different "taste" of the
different communities?

I could make (up) a lot of arguments in this debate: One could argue a lot about
which way is "cleaner", which way is more convenient to write, which way is more
readable, which way helps the reader more in understanding the code, and which
way is nudging developers to write better tests.

All these arguments would miss the point:

The difference in setting up test structure is not made based on these
arguments, but it rather reflects the paradigmatic differences between the
languages. The biggest reason for keeping tests separated from source code in
Python and JavaScript is that it is easier to exclude them from the deployment
that way.

In Rust, this reason does not matter. While Python, Ruby, and JavaScript are
interpreted languages, Rust is a compiled language, and the compiler makes sure
that test functions do not end up in the compiled binary. If we wanted to
achieve the same in Python, it would require an extra build step which
essentially removes the test code. This would require an additional dependency,
it could be a source of bugs, or it could mess with static code analysis. In
effect, in interpreted languages, it less advisable to have the tests and the
code mixed, if they are supposed to be excluded from the deployment.


# How to Deal with Lack of Orientation

Those two were examples of how we can be blinded by the paradigm we live in.
Once we have established a habit based on the paradigm we work in, we can come
up with a lot of post-rationalized justification for that habit. If one is a
person who tries to justify what they are doing, it seems almost inevitable to
run into this fallacy.

When we change into a different paradigm, we are given the chance to question
our reasoning there. That is the optimistic version. In my experience, entering
new contexts was sometimes frustrating and irritating for me. A new paradigm and
the lack of orientation it comes with is scary. That is why I would like to give
my past self some advice:

1. Notice whenever you feel strong judgment about something. When terms such as
   "strange", "wrong", "foreign", "dirty", etc. come to mind, understand that
   this can be a hint that you fail to see some paradigmatic differences.

2. Ask people for help. Instead of trying to convince others of your idea of
   how things should be organized, try to get someone to explain the paradigm to
   you.

3. Ask questions that help you understand the context and explore the
   paradigmatic implications, such as:
    + What other decisions is this influenced by?
    + What other decisions does this influence?
    + When was this decided, what is the history of this decision?
    + Who decided this?

Asking those questions does not mean that you are wrong. Maybe it turns out that
your opinion about how things should be organized is still going to be standing.
You can still decide to stick to your judgement. But this approach will help you
rule out some of the scenarios where your judgement is based on a lack of
understanding of the situation you are in, and is only a reflection of your
anxiety and helplessness.
