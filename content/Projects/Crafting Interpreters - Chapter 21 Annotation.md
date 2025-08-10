---
tags:
  - project/annotation
  - release
date created: 2025-01-18

status: stable
---
# Intro - Globals
We implement Globals in this chapter. 

As a reminder, global variables are lazily evaluated - they are resolved dynamically at runtime. This frees us up to be able to define global variables *after* they are referenced by other code.

Local variables do not follow this rule. 

As global variable declarations are statements, we now have to implement statement support. Recall that statements can be split into “declarations”, which effectively assign a value to a name, and “proper statements”, which include control flow, loops, and built-in operations like `print`.

# Variable Declarations 
Implementing variable declarations requires implementing three primitive operations:
- declaring a new variable using `var`;
- accessing the value of a previously defined variable using an identifier;
- overwriting the value of a variable using assignment.

Note that in `clox` land, a variable’s “name” or identifier is its index in the array representing our constant pool for the given code chunk. The index and the variable’s value are stored as key-value pairs in a hash table for all global variables. 

From the user’s standpoint (in Lox land), the variable’s name or identifier is its lexeme. The lexeme is stored in the constant pool at the index previously specified.

# Handling precedence for assignment
Normally, assignment is a low-precedence operator. But we want to disallow scenarios like these:

```sml
a + b = c + d
```

`a + b` is not a valid assignment target. But if we don’t implement a special rule to tell our parser that, it will assume that it can parse the ` = ` like any other infix operator in the precedence ordering. 

We have to require that the parser parse ` = ` only if it occurs in the context of a low-enough precedence expression to allow assignment. We do this with a flag, `canAssign`.

Assignment is already the lowest-precedence expression, so this basically means we allow assignment if:
- we’re already parsing an assignment. This allows chained assignment, e.g. `a = b = 3`. Or;
- we’re parsing a top-level expression like an expression statement.

# Design Miscellany 
- As before, we disallow declaring variables directly inside the body of a control flow statement. 

- Languages often supply multiple implementations for the same feature in different use cases to optimize compiler performance. 

`meta:` Throughout your [emulative](<Op emulate.md>) journey you must remember that single-pass compilation might well be a simple and efficient-enough method to parse source code from many simple languages. But it’s a bad way to learn. Multi-pass compilation is the nature of life. Don’t pretend you’re a simple compiler needing to understand one thing before moving on the next and screaming some kind of runtime error if you don’t understand something. The mind is nonlinear and processes information nonlinearly, over multiple passes, and contextually by approaching information from multiple angles. 

`debugging 01.28`
Changed line 2 of `allocateString`:
 ```c
ObjString* string = ALLOCATE(ObjString, 1);
```
 to 
 ```c
ObjString* string = ALLOCATE_OBJ(ObjString, OBJ_STRING); 
```
which allowed strings to be properly allocated and loaded onto the call stack.

