**************************
Frequently Asked Questions
**************************

What are the differences between Agda and Idris?
================================================

The main difference is that Idris has been designed from the start to support
verified systems programming through easy interoperability with C and high
level language constructs to support domain specific language implementation.
Idris emphasises general-purpose programming, rather than theorem proving, and
as such includes higher level programming constructs such as type classes and
do notation. Idris also supports tactic based theorem proving, and has a
lightweight Hugs/GHCI style interface.

Is Idris production ready?
==========================

Idris is primarily a research tool for exploring the possibilities of software
development with dependent types, meaning that the primary goal is not (yet) to
make a system which could be used in production. As such, there are a few rough
corners, and lots of missing libraries. Nobody is working on Idris full time,
and we don't have the resources at the moment to polish the system on our own.
Therefore, we don't recommend building your business around it!

Having said that, contributions which help towards making Idris suitable
for use in production would be very welcome - this includes (but is not
limited to) extra library support, polishing the run-time system (and ensuring
it is robust), providing and maintaining a JVM back end, etc.

Why does Idris use eager evaluation rather than lazy?
=====================================================

Idris uses eager evaluation for more predictable performance, in particular
because one of the longer term goals is to be able to write efficient and
verified low level code such as device drivers and network infrastructure.
Furthermore, the Idris type system allows us to state precisely the type
of each value, and therefore the run-time form of each value. In a lazy
language, consider a value of type ``Int``:

.. code-block:: idris

    thing : Int

What is the representation of ``thing`` at run-time? Is it a bit pattern
representing an integer, or is it a pointer to some code which will compute
an integer? In Idris, we have decided that we would like to make this
distinction precise, in the type:

.. code-block:: idris

    thing_val : Int
    thing_comp : Lazy Int

Here, it is clear from the type that ``thing_val`` is guaranteed to be a
concrete ``Int``, whereas ``thing_comp`` is a computation which will produce an
``Int``.

How can I make lazy control structures?
=======================================

You can make control structures using the special Lazy type. For
example, ``if...then...else...`` in Idris expands to an application of
a function named ``ifThenElse``. The default implementation for
Booleans is defined as follows in the library:

.. code-block:: idris

    ifThenElse : Bool -> (t : Lazy a) -> (e : Lazy a) -> a
    ifThenElse True  t e = t
    ifThenElse False t e = e

The type ``Lazy a`` for ``t`` and ``f`` indicates that those arguments will
only be evaluated if they are used, that is, they are evaluated lazily.

Evaluation at the REPL doesn't behave as I expect. What's going on?
===================================================================

Being a fully dependently typed language, Idris has two phases where it
evaluates things, compile-time and run-time. At compile-time it will only
evaluate things which it knows to be total (i.e. terminating and covering all
possible inputs) in order to keep type checking decidable. The compile-time
evaluator is part of the Idris kernel, and is implemented in Haskell using a
HOAS (higher order abstract syntax) style representation of values. Since
everything is known to have a normal form here, the evaluation strategy doesn't
actually matter because either way it will get the same answer, and in practice
it will do whatever the Haskell run-time system chooses to do.

The REPL, for convenience, uses the compile-time notion of evaluation. As well
as being easier to implement (because we have the evaluator available) this can
be very useful to show how terms evaluate in the type checker. So you can see
the difference between:

.. code-block:: idris

    Idris> \n, m => (S n) + m
    \n => \m => S (plus n m) : Nat -> Nat -> Nat

    Idris> \n, m => n + (S m)
    \n => \m => plus n (S m) : Nat -> Nat -> Nat

I have an obviously terminating program, but Idris says it possibly isn't total. Why is that?
=============================================================================================

Idris can't decide in general whether a program is terminating due to
the undecidability of the `Halting Problem
<https://en.wikipedia.org/wiki/Halting_problem>`_. It is possible, however,
to identify some programs which are definitely terminating. Idris does this
using "size change termination" which looks for recursive paths from a
function back to itself. On such a path, there must be at least one
argument which converges to a base case.

- Mutually recursive functions are supported
- However, all functions on the path must be fully applied. In particular,
  higher order applications are not supported
- Idris identifies arguments which converge to a base case by looking for
  recursive calls to syntactically smaller arguments of inputs. e.g.
  ``k`` is syntactially smaller than ``S (S k)`` because ``k`` is a
  subterm of ``S (S k)``, but ``(k, k)`` is
  not syntactically smaller than ``(S k, S k)``.

If you have a function which you believe to be terminating, but Idris does
not, you can either restructure the program, or use the ``assert_total``
function.

When will Idris be self-hosting?
================================

It’s not a priority, though not a bad idea in the long run. It would be a
worthwhile effort in the short term to implement libraries to support
self-hosting, such as a good parsing library.

Does Idris have Universe Polymorphism? What is the type of ``Type``?
====================================================================

Rather than Universe polymorphism, Idris has a cumulative hierarchy of
universes; ``Type : Type 1``, ``Type 1 : Type 2``, etc.
Cumulativity means that if ``x : Type n`` then also ``x : Type m``,
provided that ``n <= m``.

Why does Idris use ``Float`` and ``Double`` instead of ``Float32`` and ``Float64``?
===================================================================================

Historically the C language and many other languages have used the
names ``Float`` and ``Double`` to represent floating point numbers of
size 32 and 64 respectivly.  Newer languages such as Rust and Julia
have begun to follow the naming scheme described in `IEE Standard for
Floating-Point Arithmetic (IEEE 754)
<http://en.wikipedia.org/wiki/IEEE_floating_point>`_. This describes
single and double precision numbers as ``Float32`` and ``Float64``;
the size is described in the type name.

Due to developer familiarity with the older naming convention, and
choice by the developers of Idris, Idris uses the C style convention.
That is, the names ``Float`` and ``Double`` are used to describe
single and double precision numbers.

What is -ffreestanding?
=======================

The freestanding flag is used to build Idris binaries which have their
libs and compiler in a relative path. This is useful for building binaries
where the install directory is unknown at build time. When passing this
flag, the IDRIS_LIB_DIR environment variable needs to be set to the path
where the Idris libs reside relative to the idris executable. The
IDRIS_TOOLCHAIN_DIR environment variable is optional, if that is set,
Idris will use that path to find the C compiler.

Example::

   IDRIS_LIB_DIR="./libs" IDRIS_TOOLCHAIN_DIR="./mingw/bin" CABALFLAGS="-fffi -ffreestanding -frelease" make

What does the name ‘Idris’ mean?
================================

British people of a certain age may be familiar with this
`singing dragon <https://www.youtube.com/watch?v=G5ZMNyscPcg>`_. If
that doesn’t help, maybe you can invent a suitable acronym :-) .

Where can I find more answers?
==============================

There is an `Unofficial FAQ
<https://github.com/idris-lang/Idris-dev/wiki/Unofficial-FAQ>`_ on the wiki on
github which answers more technical questions and may be updated more often.
