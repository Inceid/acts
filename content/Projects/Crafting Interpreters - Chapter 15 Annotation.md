---
tags:
  - project/annotation
  - release

status: stable
---
In this chapter, we build a **Virtual Machine** (VM), which takes bytecode and runs it.

The key functionality required for the VM to run a user’s code is that, given a numeric opcode (as bytecode), it needs to **dispatch** or **decode** the instruction to the right C code that implements the instruction’s semantics.

When executing operations, we follow the same convention as in `jlox`: recursively completely evaluate the left argument, then the right, then perform the operation. Note, however, that this behavior takes a special form here. When we are done evaluating a left operand at any level of recursion, we have to keep its value around as a temp variable until we have the right value to consume both with their operator. Because we evaluate the left first, it is the last one to be consumed.
First in, last out. The way we track temp values naturally behaves like a stack!

In summary: now the basic structure of our VM is complete. Given a stream of user code already presented as bytecode, the VM reads the stream into chunks, processes the series of bytes into instructions, and executes those instructions in C. 
Along the way, it keeps track of its state using an instruction pointer `ip`, the current chunk, and a stack for scheduling execution. For each chunk being executed, we have the chunk’s constant values, bytecode instructions and operands, and line information for error reporting.

The stack is key. “It acts like a shared workspace that \[instructions\] can all read from and write to” (281) - no awareness of the environment or closure needed!

# Design Miscellany 

- Since we modify the location of the current instruction `ip` so much, we use a pointer `uint8_t*` to represent it. It’s faster to dereference a pointer than looking up a value in, say, an array.