# Designing programming languages with IDEs in mind.

In this post we'll discuss how having the way users write code in mind affects
the design of programming languages. We will start with a feature called
[named parameters]((https://en.wikipedia.org/wiki/Named_parameter))
as an example.

## Named parameters

Smalltalk is a programming language with named parameters.
We'll explain what that means with an example:

* Smalltalk code looks like this:
  `Rectangle left: 0 right: 10 top: 100 bottom: 200`
* The equivalent in
  [C](https://en.wikipedia.org/wiki/C_\(programming_language\))
  looks like this:
  `NSMakeRect (0, 100, 10, 100)`

[Smalltalk](https://en.wikipedia.org/wiki/Smalltalk)
(late 1970s) was designed to be used via its
[IDE](https://en.wikipedia.org/wiki/Integrated_development_environment),
which was released with it
(an editor specifically designed to edit code in Smalltalk).

The C code can be written on a whiteboard more quickly than Smalltalk,
but writing them on the computer takes the same effort -
when using Smalltalk's editor the argument names were auto-completed.

Which version of the code is easier to read? We're favoring Smalltalk.
What does each of the four variable in C mean?
Left? Right? Width? Height? Center positions?
Only a programmer already familiar with that library function
will know what they mean without needing to have a look at the documentation.
This makes C require a steeper learning curve than Smalltalk,
where the named parameters make the code
[self-documenting](https://en.wikipedia.org/wiki/Self-documenting).

### Right vs right

We believe Smalltalk's designers made
the right design choice for function call syntax,
but that the designers of C have also made the right design choice.

How can that be?
When C was released in the early 1970s people used
generic text editors to edit C code.
And typing the parameter names in a generic text editor does take more effort.

Programming language designers should have their users and the environment
that they are going to use in mind when designing the language.
C's designers made the right choice
given that their users used generic text editors
(IDEs were not prevalent at the time).

### IDEs helping after the fact

IDEs certainly help editing C and its offspring languages
([C++](https://en.wikipedia.org/wiki/C%2B%2B),
[C#](https://en.wikipedia.org/wiki/C_Sharp_\(programming_language\)),
[Java](https://en.wikipedia.org/wiki/Java_\(programming_language\)),
[JavaScript](https://en.wikipedia.org/wiki/JavaScript),
and more).

Some editors display popups when hovering on function calls
to show the function's documentation.
That's certainly useful, but the Smalltalk reading experience
is still superior, as one can simply glance with their eyes instead of moving
the mouse and reading what pops up.
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

## More design choices

After discussing named parameters as an example to the general concept,
we'll briefly present more language design choices we made for Lamdu,
given the different design trade-offs present when we design the language to
be used with its dedicated reference IDE.

### Explicit lazy evaluation

Again, we'll explain what this means with a code example.

* Lamdu: `x % 3 == 0 || ◗ x % 5 == 0`
* [Haskell](https://www.haskell.org):
  `x % 3 == 0 || x % 5 == 0` (same as Lamdu, but without the `◗` symbol)
* C++: `x % 3 == 0 || x % 5 == 0` (looks exactly the same as Haskell)
* Python: `x % 3 == 0 or x % 5 == 0`
* Smalltalk: ``(x \\ 3) = 0 or: [(x \\ 5) = 0]``

In all examples above, the
["logical-or"](https://en.wikipedia.org/wiki/Logical_disjunction)
operator (`||`/`or`),
[short-circuits](https://en.wikipedia.org/wiki/Short-circuit_evaluation).
That means that the computation on its right hand side does not get evaluated
if the expression on its left hand side evaluates to `True`.

First let's describe the difference between the Haskell and C++ examples above,
which look exactly the same:
* In C++ the `||` operator is a built-in language primitive
  and such operators cannot be defined by users and libraries.
  In C, user-defined operators do not support short-circuiting
  (The same holds for Python).
* In Haskell, the `||` operator is defined in the standard library and can be
  reimplemented by users. Haskell supports this because it has
  "pervasive lazyness" - this means that expressions only get evaluated
  when their values are needed.
  Instead of getting the values of the arguments to a function, it actually
  gets ["thunks"](https://en.wikipedia.org/wiki/Thunk),
  which can be evaluated to get the value when and if it is necessary.

While Haskell's use of lazy evaluation allows
defining control structures in the library,
this feature doesn't come free of disadvantages,
and that's why this feature isn't common in programming languages.

For Lamdu we made a different choice, which also happens to be the same choice
made in Smalltalk: support lazy evaluation, but not automatically everywhere.

The logical-or operator in Lamdu gets a boolean on its left side, and a
"deferred computation" (aka function with no arguments)
of a boolean on its right side.
The syntax for these computations is very lightweight -
the `◗` symbol preceding the computation,
with parenthesis for disambiguation when necessary
(the Smalltalk syntax is square brackets around the expression).

This choice does not make writing code harder -
the editor completes the pattern of `_ || ◗ _` automatically.

### Ad-hoc records

In most statically types languages,
[records/structs](https://en.wikipedia.org/wiki/Record_(computer_science))
need to be declared "ceremoniously" before they are used,
and to avoid this effort for one-off uses, some languages also support
anonymous tuples (`(a, b)` in Haskell, `std::pair<a, b>` in C++).
