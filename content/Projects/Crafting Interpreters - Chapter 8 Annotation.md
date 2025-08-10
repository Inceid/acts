---
tags:
  - project/annotation
  - release
date created: 2024-11-22

status: stable
---
# Intro
Now, we implement **bindings**, which form an essential component of software. This requires a notion of internal state: the interpreter has to maintain a set of names referring to values for the duration of a program's execution in its "working memory". 
We implement this by adding support for **statements**, which are segments of code that produce side effects. For our code, this mostly comprises `print` and `var` statements, with expressions to access and assign variables. 

Separating base class implementations for `Expr` and `Stmt` allows us to avoid errors that involve passing `Expr`'s where `Stmt`'s are expected and vice versa. For Lox, wherever you have an expression, you cannot also have a statement.

To implement the `Stmt` class, we modify the `GenerateAst` script we used to create our code for the `Expr` class to generate code for `Stmt`.
# Assignments
To implement assignments, we use a nifty trait of the variable assignment structure: the lefthand side of an assignment is always a valid expression. Take
>`newPoint(x + 2, 0).y = 3;`

for example. The lefthand side `newPoint(x + 2, 0).y` is also a valid expression, as it evaluates to the `y` value of the `newPoint` call. This means we can parse the lefthand side of an assignment as if it were an expression, then return a syntax tree that turns it into an assignment target. If it isn't a valid target, report a syntax error.
# Scope
Finally, we implement **scope**. Scope defines a region where a name maps to a value or object. Scope allows the same name to refer to different objects in different contexts. In Lox and most languages, variables are **lexically scoped** (or statically scoped) - you can infer the scope from the text of the code.

For methods and fields on objects, as well as functions Lox uses **dynamic scoping** - here, we don't know what a name refers to until runtime. Scope in Lox is explicitly controlled by curly-braced blocks.

Importantly: 
* when entering a block, we save the environment from the previous (or global) code block and spawn a fresh one, accumulating variables as we go. At the end of the block, we delete it and restore the old environment. This allows us to ensure that different blocks don't interfere with each other or the global environment.
* global variables are passed into local scopes for use as needed. 

We implement both ideas by chaining environments together: each inner environment is "linked" to the environment immediately one level higher in scope.
* To look up variables in an outer scope, the interpreter will have to walk upwards until it finds that variable, reporting the usual errors if it fails.
`portal:` [clox Locals](<Crafting Interpreters - Chapter 22 Annotation>)
# `par:` Design decisions of note
* p. 121 For variable scoping practices, "when in doubt, do what Scheme does". Scheme was invented to introduce the idea of variable scoping to the world.
* to let us play with the interpreter as we're building it, we build `print` into the language rather than as a library.
* we disallow variable declarations within control flow statements in the vein of Java and C.
	* generalization: For variable declarations, we require that declarations have a higher precedence than all other statements, so that we can enforce declarations appearing only in some sections of code and not others.
* we implement undefined variable errors as runtime errors. Implementing them as syntax errors would pose problems for recursively defined functions, especially for mutually recursive functions.