---
tags:
  - project/annotation
  - release

status: stable
---
# Bytecode 
Now, we reimplement `lox` with a compiler in `C`.

A comparison of our tree-walk AST-based interpreter and a compiler that translates user code into bytecode justifies the move to the latter for sake of efficiency and scalability:

| Tree-walk interpreter | Compiler to native code | Compiler to Bytecode |
| --------------------- | ----------------------- | -------------------- |
| Simple                | Complex-est             | Complex              |
| Portable              | Platform-specific       | Portable             |
| Slow                  | Fastest                 | Fast                 |

Bytecode represents a simplified form of machine code. Relatively speaking, it's the easiest form of machine architecture code to write in. However, it doesn't correspond to any real chip language. To use bytecode, we write a **Virtual Machine**, a simulated chip implemented in software that interprets or executes the bytecode. Most, if not all, chips and operating systems support conversion of C to native code, so writing our VM in C ensures portability.

In this chapter, we will create the basic skeleton of `clox` and write enough data structures to be able to represent bytecode. 

Code will be represented as chunks of bytecode, where each instruction is represented as one byte (called the instruction's **opcode**). 

---

# Disassembly 
Once we’ve figured out the basic of creating, representing, and freeing chunks, we need to write a **disassembler** that converts a chunk into an unambiguous string representation of its instructions (like in [Chapter 5](<Crafting Interpreters - Chapter 5 Annotation>) when we implemented `AstPrinter.java` to pretty-print AST nodes).

The basic idea: for each byte of opcode in a chunk, we switch on the instruction type. For each switch case, we dispatch to a utility function that handles stringifying it (251).

# Values 
We represent values as `double` floats for now. Critically, we introduce an abstraction layer between the code that represents values and the code that manipulates/interacts with values. This allows us to change our representation (by changing the `typedef` for values) without changing the rest of the code that uses values. 

Following Java, for each chunk we store its values in a **constant pool**, an dynamic array specialized to store values. 

# Errors
When a runtime error occurs, we return the line number of the instruction where the error occured. The line information is contained in each chunk in an array that exactly parallels the bytecode array.

In summary, we have reduced ASTs to three dynamic arrays packaged in a chunk: an bytecode array, constant pool, and a line number array. This reduction is what allows the interpreter to work so much faster than `jlox`: array indexing and modification is much faster than tree traversal!

# Design Miscellany 
Recall C's ternary syntax for conditionals:
```
condition ? value_if_true : value_if_false
```
