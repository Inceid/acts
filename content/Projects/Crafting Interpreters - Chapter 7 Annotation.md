---
tags:
  - project/annotation
  - release
date created: 2024-11-17

status: stable
---
In Lox, values are created by literals, computed via expressions, and stored in variables.
We can use `java.lang.Object` to represent dynamically typed variables that admit of different possible representations at runtime. We can do runtime typechecking using the `instanceof` operation.
* Note: we need the `instanceof` operation to disambiguate polymorphic operations like `+`, which could represent string concatenation or numerical addition. `instanceof` dynamically typechecks each operand at runtime to select the appropriate behavior of `+`. 

Important terminological distinctions:
- a literal is a bit of syntax that produces a value when evaluated by the parser, and which *always* appears in the source code.
- a value is a primitive object produced by evaluating an expression. It only exists at runtime.

!Note: The core of what makes a language dynamically typed is the ability to evaluate types to the specificity required *at runtime*, and keep them polymorphic otherwise.
As we have currently implemented it, our interpreter is doing a **post-order traversal** of its given expression syntax tree: each node evaluates its children before applying its operator to the result.

Importantly, we need to add a layer of error handling that checks for errors that occur at runtime.
The difference between what we're doing here and in Chapter 6 (where we implemented error handling for syntax) is that these errors are detected and reported *while the code is running*, as they indicate a problem not with the code's syntax, but with its semantics (or internal logic per the grammar and typechecking rules of the language's operations).
Best practices for this usually involve reporting the error, exiting execution of the rest of the erroneous code tree, but keeping the interpreter alive for the user to continue to use (that is, not exiting the stack and shutting everything down entirely). 

To hook up the interpreter to the main `Lox` class that calls it, we use a single function `interpret()` which takes an `Expr`, calls `evaluate()` (which passes the expression to the visit methods to interpret it or throw an error back to `Lox`), returns the result if the tree is semantically valid, and catches and reports any runtime errors back to the user otherwise.
* `analog` For our Java Lox implementation, think of `interpret()` as the face man, the CEO of the Interpreter sector. It reports directly to the public Lox class, and hence forms the Interpreter's first line of communication with the rest of the program.

`par:`
* Why do we overload certain operators? What design considerations are there for ambiguating certain operators over others?
* Strings have concatenation. Why don't they have truncation and other manipulations as overloaded infix operators?
* Re: p. 107, Why do we provide a unique exit code to the main calling process when a runtime error occurs? What is 'shell etiquette'?