---
tags:
  - project/annotation
  - release
date created: 2025-01-28

status: stable
---
We implement control flow. 

Control flow operations revolve mostly around modifications of the instruction pointer `vm.ip`, which indicates our exact current position in the user’s code - the next instruction to execute. 

By default, the instruction pointer marches forward instruction by instruction. But control flow operations allow it to jump around the code as it pleases. 

One problem we have to solve is the following: given code 
```sml
if (expr()) {
	printf("do this");
	... other instructions ...
} else {
	printf("do that");
}
```

when compiling this to bytecode, we use an operation `OP_JUMP_IF_FALSE` to jump over the `then` portion of the `if` branch if the predicate `expr()` is false. We have to load `OP_JUMP_IF_FALSE` onto the stack with an operand - the number of bytes to jump ahead. But how do we know how far to jump before we’ve read the rest of the code?

We use **backpatching** to solve this. The idea: emit the jump with a placeholder operand. Read the rest of the `then` body, keeping a count of the number of bytes passed. Then go “back” and “patch” the placeholder with the real value. 

Within the compiler, this looks structurally something like this (as an example):
```c
static void ifStmt() {)
	consume(TOKEN_LEFT_PAREN, "Expect '('!");
	expression(); // evaluate a boolean predicate
	consume(TOKEN_RIGHT_PAREN, "Expect '('!");

	int jump = emitJump(OP_JUMP_IF_FALSE);
	// code we want to skip over ...
	patchJump(jump);
	emitByte(OP_POP); // add this if you want to pop the predicate's value
}
```

Functionally, this code encapsulates the basic patterns of control flow using jump operations. It:
1. evaluates a predicate and pushes it onto the stack;
2. emits an `OP_JUMP_IF_FALSE` instruction with a dummy operand offset of bytes to jump over;
3. emits a `patchJump` instruction to update the dummy operand to the actual offset to jump over;
4. pops the predicate’s truth value from the stack (unless it’s needed in further computation steps).

Implementing for loops is a little more complex. The summary: we know there are three (optional) clauses: a variable (or expression) initializer, a condition to stop the loop, and an incrementer operation. 
1. We parse the initializer clause just like a variable declaration or expression statement.
2. We parse the condition clause as an expression with a jump to the end of the loop if the condition is false. 
1. We parse the incrementer clause as follows:
	* when we see an incrementer clause, we jump over it to the loop body and execute the body.
	- we record our “jump back” offset to the location of the incrementer expression. 
	- we emit the incrementer expression.
	- we jump back to the condition clause to start another iteration.

# Design Miscellany 
By design, we ensure that we pop a condition’s value before executing the chosen branch for that condition. 

`meta:` Dynamically typed programs are harder to reason about - in general, dynamic structures are harder to reason about than static structures. But why the focus on reasoning about programs? If you’re good enough at reasoning over code, why not try madness over code?