# Designing programming languages with IDEs in mind.

In this post we'll discuss how having the way users write code in mind affects
the design of programming languages,
and how it affected our own decisions when designing the language for Lamdu.

We will start by discussing a feature called
[named parameters]((https://en.wikipedia.org/wiki/Named_parameter))
as a first example.

## Named parameters

[Smalltalk](https://en.wikipedia.org/wiki/Smalltalk)
is a programming language with named parameters.
Here's an example to illustrate what that means:

| Language  | Example Code
|-----------|-----------------------------
| Smalltalk | `Rectangle left: 0 right: 10 top: 100 bottom: 200`
| C         | `NSMakeRect (0, 100, 10, 100)`

Smalltalk was designed to be used via its
[IDE](https://en.wikipedia.org/wiki/Integrated_development_environment),
which was released with it
(an editor specifically designed to edit code in Smalltalk).

The [C](https://en.wikipedia.org/wiki/C_\(programming_language\))
code can be written on a whiteboard more quickly than Smalltalk,
but writing the code on the computer takes the same effort for both languages -
when using Smalltalk's editor the argument names are auto-completed.

Which version of the code is easier to read? We're favoring Smalltalk.
What does each of the four variable in C mean?
Left? Right? Width? Height? Center positions?
Only a programmer who is already familiar with that library function
will know what they mean without needing to have a look at the documentation.
This makes C require a steeper learning curve than Smalltalk,
where the named parameters make the code
[self-documenting](https://en.wikipedia.org/wiki/Self-documenting).

### Right vs right

We believe that Smalltalk's designers made
the right design choice for function call syntax,
but that the designers of C have also made the right design choice.

How can that be?
When C was released in the early 1970s people used
generic text editors to edit C code.
And typing the parameter names in a generic text editor does take effort.

Programming language designers should have their users and the environment
that they are going to use in mind when designing the language.
C's designers made the right choice
given that their users used generic text editors
(IDEs were not prevalent at the time).

### IDEs helping after the fact

IDEs certainly help editing C and other languages deriving from it
([C++](https://en.wikipedia.org/wiki/C%2B%2B),
[C#](https://en.wikipedia.org/wiki/C_Sharp_\(programming_language\)),
[Java](https://en.wikipedia.org/wiki/Java_\(programming_language\)),
[JavaScript](https://en.wikipedia.org/wiki/JavaScript),
and more).

Some editors display popups when hovering over function calls
to show the function's documentation.
That's certainly useful, but the Smalltalk reading experience
is still superior, as one can simply glance with their eyes instead of moving
the mouse pointer and reading what pops up.
This also makes for an easier pair/group programming experience,
as programmers not holding the mouse still get the relevant info.

IDEs can be better than generic text editors,
but an IDE and a language that was designed for one
can make for an even better experience.

### Notable programming languages with named parameters

* [Objective-C](https://en.wikipedia.org/wiki/Objective-C):
  Released with an IDE.
* [SWIFT](https://en.wikipedia.org/wiki/Swift_\(programming_language\)):
  Supports both conventions, also released with an IDE
  (The SWIFT term is
  [External Parameter Names](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Functions.html#//apple_ref/doc/uid/TP40014097-CH10-ID167)).
* [Python](https://www.python.org) supports both conventions and leaves the
  function's caller the choice of which convention to use
  (The Python term is
  [keyword arguments](https://docs.python.org/3.5/glossary.html#term-argument)).

### Lamdu's angle

In Lamdu we decided to have named parameters, and similarly to SWIFT,
a function's definition determines how the function calls should be displayed.

In Lamdu:

* A function with a single parameter don't use named parameters.
  If there is only one parameter name then
  the function name would already make its meaning clear.
* A function with two parameters may be an infix operator.
* A function with more than one parameter may either have the
  "Verbose" presentation mode (named parameters),
  or be verbose on all function arguments except the first one ("OO" mode).

## More language design choices with the IDE in mind

After discussing named parameters as an example to the general concept,
we'll briefly present more language design choices we made for Lamdu,
given the different design trade-offs present when we design the language to
be used with its dedicated reference IDE.

### Explicit lazy evaluation

Again, we'll explain what this means with a code example.

| Language                           | Example Code
|------------------------------------|-----------------------------
| Lamdu                              | `x % 3 == 0 || ◗ x % 5 == 0`
| [Haskell](https://www.haskell.org) | `x % 3 == 0 || x % 5 == 0`
| C/C++                              | `x % 3 == 0 || x % 5 == 0`
| Python                             | `x % 3 == 0 or x % 5 == 0`
| Smalltalk                          | `(x \\ 3) = 0 or: [(x \\ 5) = 0]`

In all examples above, the
["logical-or"](https://en.wikipedia.org/wiki/Logical_disjunction)
operator (`||`/`or`)
[short-circuits](https://en.wikipedia.org/wiki/Short-circuit_evaluation).
That means that the computation on its right hand side does not get evaluated
if the expression on its left hand side evaluates to `True`.

First let's describe the difference between the Haskell and C++ examples above,
which on the surface look exactly the same:
* In C++ the `||` operator is a built-in language primitive
  and such operators cannot be defined by users or libraries.
  In C++, user-defined operators and functions do not support short-circuiting
  (The same holds for Python).
* In Haskell, the `||` operator is actually defined in the standard library
  (it can be implemented by users). Haskell supports this because it has
  "pervasive lazyness" - this means that expressions only get evaluated
  when their values are needed.
  Instead of getting the values of the arguments to a function, it actually
  gets ["thunks"](https://en.wikipedia.org/wiki/Thunk),
  which can be evaluated to get the value if and when it is necessary.

While Haskell's use of lazy evaluation allows
defining control structures in the library,
this feature doesn't come free of disadvantages,
and that's why it isn't a common feature in programming languages.

For Lamdu we made a different choice, which also happens to be the same choice
made in Smalltalk: support lazy evaluation, but not implicitly and everywhere.

The logical-or operator in Lamdu gets a boolean on its left side, and a
"deferred computation" (aka function with no arguments)
of a boolean on its right side.
The syntax for these computations is very lightweight -
the `◗` symbol preceding the computation,
with parenthesis for disambiguation when necessary
(the Smalltalk syntax for this is square brackets around the expression).

This choice does not make writing code harder -
Lamdu's editor completes the pattern of `_ || ◗ _` automatically.

### Records vs tuples

Let's again start with an example.
Let's say we have a function that performs a calculation and returns two values:
"speed" and "azimuth".

In most programming languages, we may either return an anonymous tuple
`(speed, azimuth)` or use a
[record type](https://en.wikipedia.org/wiki/Record_(computer_science))
for them. Each approach has its own advantages.

Advantages of tuples:
* We don't need to remember the exact name of the field -
  whether it was "azimuth" or maybe "angle"?
  An IDE would solve this problem by offering field name completions.
* We avoid the laborious type declaration ceremony.
  In a language which has a
  [structual type system](https://en.wikipedia.org/wiki/Structural_type_system)
  with [type inference](https://en.wikipedia.org/wiki/Type_inference),
  type declarations are not necessary.

Advantages of records:
* We don't need to remember the order of the values,
  so there's no risk of confusing them.
* In statically types languages, using records makes programs more reliable by
  making better use of the type system
  for distinguishing different things with types,
  and also provides more info about functions in their
  [type signatures](https://en.wikipedia.org/wiki/Type_signature).

In Lamdu, the IDE and type system make
the advantages of tuples available for records.
Records in Lamdu have the reliability and readability of records
with the the ease and convenience of tuples.

Thus, tuples are no longer necessary so Lamdu only has records
(tuples can still be implemented as degenerate records like C++'s `std::pair`).

## Conclusion

Having an IDE affects the trade-offs of programming language design.

In Lamdu's case, as our aim is to create a "next-gen" IDE,
and as no programming language was yet designed for such an IDE,
we've decided to design a new language,
resembling an existing language (Haskell)
but with some modifications due to the different design trade-offs given the
different environment users will use to write the code.
