---
tags:
  - project/annotation
  - release
date created: 2024-11-22

status: stable
---

In this chapter, we implement **Turing-completeness**. Informally, a Turing-complete language (or system) is any language that is expressive enough to do what a Turing machhine can do, which is compute any computable function. 

> The key test is this: if you can use your language to successfully simulate a Turing machine, it can compute any computable function. It is therefore **Turing-complete**.

What this essentially amounts to is a system that includes arithmetic, control flow, and the ability to allocate and use (theoretically) arbitrary amounts of memory. We will implement the second condition in this chapter.

Control flow divides into **conditional** flow, which skips or deliberately does not execute some chunk of code, and **looping** flow, which executes a chunk of code more than once.

Conditional flow jumps forward; looping flow jumps back and repeats.

There's a classic problem with conditionals that arises when you have multiple `if`'s and a "dangling" `else` case. The question is this: which `if` does the `else` belong to?
Most languages choose the same (arbitrary) disambiguation: the `else` is bound to the nearest `if` preceding it.

Lox's parser already does this, as `ifStatement()` eagerly looks for an `else` before returning. This means the innermost `if` will claim the nearest `else` before the outer `if`'s can get to it.

For `while` loops, we implement them by noticing that the condition of a `while` is an expression, which, while evaluated to true, executes a statement in the body of the loop.

`for` loops are a bit more complicated. They contain up to three loop clauses, which consist of a variable declaration called an *initializer*, the *condition* to control when to exit the loop, and the *increment* to do some work at the end of each iteration, the result of which is discarded unless stored as a side effect; then a statement executed as the body of the loop. All of these clauses are optional.

> Note that Lox doesn't actually need for loops as a separate construct in the langauge - you can construct them using a regular incrementer initialization, a while loop, and a regular statement to increment the incrementer! We include it to make coding a bit more of a pleasant experience. That makes `for` loops a feature called `syntactic sugar`.

To optimize this new addition, we `desugar` the `for` loop syntax by decomposing it into more primitive forms for the back end to execute. Specifically, in `Interpreter.java`, we decompose the `for` loop into: 
* a `Block` of the initializer (if it exists), 
* the loop as a `while` loop whose condition (if it exists) is the `for` loop's condition, and 
* the count incrementer as an expression statement.
# `par:` General notes
* Control flow was quite fast to implement, and by observing, we may !Note the general flow that we seem to be going through:
	* First, extend `GenerateAst.java` with new subclasses that we want to include structures and visitor interfaces for.
	* Second, write methods in `Parser.java` for parsing the new class elements into syntax tree nodes.
		* whenever we introduce new ast classes, we fit them into the grammar wherever they belong in the precedence ordering. This creates a gap in the parser. We have to stitch those gaps by implementing parsing for these ast nodes *in a way that links the precedence ordering back together completely*.
		* furthermore, when we implement the actual parsing behavior for new asts, we make heavy use of top-down design. Namely, *we reuse and refactor as much code as possible.* 
			* example: If you already have code for evaluating an expression, and you have code for evaluating a statement, then the code for a while loop just constructs an ast node with the condition (`Expr`) and the body (`Stmt`) as processed by those functions.
	* Third, extend `Interpreter.java` with the appropriate visitor(s) for each new subclass you generated.
* Note that the way we implement conditionals, because of how Java's `if` works, allows us to potentially skip evaluating the then and else branches of the corresponding syntax tree node. If the `if` fails, it fails, and we move on.