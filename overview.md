# ebash Guide

## Overview

Bash is the ideal language of choice for writing low-level shell scripts and tools requiring direct shell access
invoking simple shell commands on a system for a number of reasons:

1. Prolific and already installed on nearly every *NIX machine out of the box which minimizes dependencies and
   installation complexity and size.
2. Extremely lightweight with very low memory and CPU requirements suitable for appliances and embedded systems.
3. Ideal for tasks involving running lots of shell commands as it is simpler than in higher level languages. This is,
   after all, bash's primary native role. As such, it does not require complex objects or frameworks for executing
   or interacting with shell commands.

That said, because bash is a lower level language, it lacks some of the features and more advanced data structures
typically found in higher level languages. As such, the goal of ebash is to provide a layer of robustness and more
advanced features on top of vanilla bash.

## Compatibility

Ebash aims for extreme compatibility with the intention that it "just works" on just about any *NIX OS. While it is not
feasible to test ebash on _every_ OS/Distro, we nonetheless test it many popular combinations on every single build. You
can view the build status any time using the build batch at the top of this README or by going here:
[Builds](https://github.com/elibs/ebash/actions?query=workflow%3APipeline+branch%3Amaster).

Here is a table showing all the currently test and validated OS/Distro
combinations:

| OS    | Disto  | Version |
| ----- | -------| ------- |
| Linux | Alpine | 3.13    |
| Linux | Alpine | 3.12    |
| Linux | Arch   | rolling |
| Linux | CentOS | 8       |
| Linux | CentOS | 7       |
| Linux | Debian | 10      |
| Linux | Debian | 9       |
| Linux | Fedora | 33      |
| Linux | Fedora | 32      |
| Linux | Gentoo | rolling |
| Linux | Ubuntu | 20.04   |
| Linux | Ubuntu | 18.04   |
| MacOS | N/A    |  11.0   |
| MacOS | N/A    | 10.15   |

## Why ebash?

### Implicit Error Detection

Bash is a strange beast.  It can be an interactive shell, in which case you’d like to be notified of failures but not
for them to be catastrophic.  And when you write scripts, it’s easy to be torn about what you want.  Sometimes you want
failures to be ignored, and other times you want to be very sure that a command doesn’t fail.

The standard bash answer to this question is that you should always check for errors if you care about them.  But
sometimes commands fail that you don’t expect to fail.  What happens when you’ve left out a test in that case because
that could never happen, right?  Or perhaps you have taken extreme care to check for errors in all possible cases.  Now
your code is wordy and more difficult to follow.

With ebash, we have decided to make scripts detect errors implicitly.  If an error occurs, you’ll detect it and your
script will blow up.  You can specifically ignore such failures when you know it’s safe, but the default is to make the
failure as obvious as possible.

This is in opposition to all bash defaults.  And, frankly, it’s a bit difficult to force bash into really blowing up on
all failures.  But we do our best with a multi-layered approach.  As long as you use the typical ebash idioms and follow
a small set of styles we prescribe, we believe that any time an error occurs in your ebash code, you’ll hear about it.

First, we use bash’s error trap functionality.  This behaves just like `set -e`, except that we get to choose what runs
when an error occurs.  Basically, we call the die function on any error.  This causes a stack trace to be generated
which will have `[UnhandledError]` in its message.  Second, we ensure that the error trap is inherited by all subshells,
because sometimes you need them, and often it’s difficult to know where there will actually be a subshell.

But sometimes bash really wants to ignore a return code.  For instance, if command substitution isn’t part of the first
command on a line, as in both of the following lines of code, bash will ignore any return code of `cmd`.

    echo "$(cmd)"
    local var=$(cmd)

But ebash enhances bash’s error detection to catch this case.  The error trap is inherited by the shell inside the
command substitution, and when it calls die, it will actually send a signal to its parent shell.  When the parent shell
receives that signal, it generates a stack trace and blows up, which is exactly what we were looking for.

However, despite the best efforts of ebash, bash allows you to shoot yourself in the foot in subtle ways that mask error
handling.  The final component of our efforts to make errors implicitly detected are some styles to avoid.
