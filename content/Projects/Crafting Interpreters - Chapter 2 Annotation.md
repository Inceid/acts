---
tags:
  - project/annotation
  - release
date created: 2024-10-26
last updated: 2025-07-16

status: stable
---
# A Map of the Territory.
A program is a list of instructions for a computer to execute. Your program is the recipe you write; the computer is the chef that executes it.
Okay, so you wrote a program and you want the computer to read and do what you say. How does it get there?

## The Process of Compilation.
Step 1: Scanning (or **'lexing'**). The computer uses a scanner or **lexer** for the language to read the words of the program. Each word is called a **token**.
Step 2: **Parsing**. The computer uses a **parser** to convert a flat sequence of tokens into an abstract syntax tree (AST), which represents the sequence in "sentence" form.
Step 3: **Static Analysis**. The computer does the following, generally in this order:
- Attaching identifiers (variable names) to their binding and scope.
- Typechecking all arguments against the code.
- Augmenting the AST with binding information, or creating a data structure to store name-value relationships between identifiers and their bindings.

>**Analogy: cs -> med**: Think of it this way: digestion in the human body happens over multiple stages, where nutrient "info" is broken down into more basic structures at each stage. Then those nutrients are integrated into your cells, keeping you alive.
>
> Program compilation is very similar. The computer is given your source code and has to break it down into syntactic and semantic ingredients, then has to integrate those semantic nutrients into its own language to run the program.

Step 4: **Intermediate Representations.** An IR acts as an interface between the source language and the computer architecture's final language (wherein the program is actually run).
- If you wanted a computer that could convert Pascal, FORTRAN, and C into x86, ARM, and SPARC (which computers should be able to do), a common intermediate representation allows you to accomplish this without having to write separate compilers for all combinations (3x3) of pairs. 
Step 5: **Optimization.** The computer uses certain rules to replace the user's source code with code that is semantically equivalent but uses as few unique operations as possible to save resources.
Step 6: **Code generation.** Now the computer converts the user's optimized code into machine code, usually into a "virtual CPU" by conversion to a language called **bytecode**. Bytecode is essentially a binary encoding of your language's lowest-level (simplest, most primitive) operations.
* Bytecode can be translated to machine code manually ahead of time, allowing for fast execution of the entire program, or...
* a **Virtual Machine** can convert the bytecode to a hypothetical chip's machine code at program runtime. This is slower because, at the last step, virtual instructions must be translated line by line, making the VM slower than the first option above.
Step 7: **Runtime.** Sometimes, you want to see information about what objects are present and what's happening to them while the program is running. You might want to modify memory while the program is running. All operations that enable this are part of the **runtime** of compilation.
#### Interpreters vs. Compilers
Interpreters run source code immediately, without compiling first. This makes them generally slower, as they have to translate source code into executable machine code line by line.
Compilers convert source code into bytecode or machine code altogether, allowing the computer to execute the entire program in its native language, at a much higher speed.