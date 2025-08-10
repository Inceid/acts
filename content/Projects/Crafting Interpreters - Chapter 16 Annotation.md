---
tags:
  - project/annotation
  - release

status: stable
---
# The Overall Structure 
Since we have a barebones VM, let’s map out the rest of the construction process.

`clox` has three phases: a scanner, compiler, and VM. 

Each pair of phases is joined by a data structure.
- Tokens from scanner to compiler;
- Bytecode chunks from compiler to VM.

In this chapter, we build the scanner.

For the scanner and throughout `clox`’s implementation, we’ll have to pay special attention to how we manage memory. C enforces disciplined memory management partly through its *very* verbose syntax and code for manipulating and modifying memory, as we saw with all the macros we needed for using dynamic arrays in [Chapter 14](<Crafting Interpreters - Chapter 14 Annotation>).

For the scanner itself, we use the same kinds of datatypes (Tokens of lexemes and line info) as in `jlox`, but this time we avoid storing lexemes as string literals so as to bypass the memory issues associated with storing and freeing those literals. Instead, we address the code stream itself.

Note that we also change our representation of user code from chunks to a character stream, similar to `jlox`.

# Keyword Scanning 
Conceptually, our method of scanning keywords works like a DFA. Starting from the first character, it reads a string character by character and transitions from one state to another until it reaches an “accept” state for a keyword. If no such state is reached by the time we reach the end of a string, it’s not a keyword.

We implement this in practice with a switch statement that handles transitions by different cases.