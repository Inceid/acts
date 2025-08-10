---
tags:
  - project/annotation
  - release

status: stable
---
# Chapter 3: The Lox Language.
`downloaded lox interpreter from craftinginterpreters.com/repo`

## Key Facts about Lox
1. Its syntax is inherited from C.
2. It is dynamically typed: variables can store values of any type, and can change types at different times. Typechecking is deferred to runtime.
3. Lox has automatic garbage collection.
4. There are only a few built-in datatypes: bools, numbers, strings, and nil (the null value).
5. It has the usual expressions.
	1. expressions followed by a `;` promote an expression to a statement.
6. For control flow, Lox uses if/else, while, and for loops just like C does.
7. Functions are first class, meaning they are real values you can get a reference to and store in variables. Lox also has **closures**: functions that hold references to surrounding variables declared in their local environment (*not within their body*).
8. It has OOP, and is a class-based language.

>we’ll name our language `indra`.