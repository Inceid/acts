---
tags:
  - project/annotation
  - release
date created: 2025-01-28

status: stable
---
# Local Variables
In this chapter we implement blocks, block scope and local variables.

Local variable scoping exactly mimics a stack in storing what variables are in scope at any given point in time. So we can use stack offsets as operands for the instructions that read and store locals.
> This means we don’t have to store them separately in the constant table to look up by name during execution! The stack and Lox’s lexical scoping rules already give us enough info to resolve local variables without needing extra memory.

In `jlox`, we used [environment chains](<Crafting Interpreters - Chapter 8 Annotation>) to track states between changing scopes. For `clox`, we again use a flat array. Each element is a local variable paired with its scope depth - the number of scopes down from the global scope that it’s declared in.

The usual churn: update the grammar → update `compiler.c`’s implementation of the grammar → 

# Three edge cases in scoping 
The same three thorny edge cases from `jlox` make a return here, and we have to handle them appropriately on top of the current implementation. 
## case 1: shadowing

given:
```sml 
var x = 3;
{
	x = 5;
	print(x);
}
```

this code should print `5` to the console. While in the inner scope, the inner `x` *shadows* or overtakes the outer `x`.

We enforce this by *always* trying to resolve assignment (set) and access (get) by looking for previously declared local variables first. We search for local variables with matching names from the innermost scope outwards. Then we default to using global get and set functions if that fails.

## case 2: declaration collision

given:
```sml
{
	var a = 3;
	var a = 5;
}
```

this code should raise an error. Re-declaring a local variable (rather than simply reassigning) is likely a user mistake.

We enforce this by walking our array of locals from the innermost scope outwards. Any time we encounter a local variable declaration with the same name in the same scope, we throw an error.

## case 3: reference before initialization
given:
```sml
{
	var a = "outer";
	{
		var a = a;
	}
}
```

this code should raise an error. We declare `a` anew in its innermost scope, yet cannot initialize it. Its initial value references itself, and it isn’t defined yet.
- Remember that we can’t use `“outer”` as the value for `a`, since the inner `a` must reference names within its current scope. 

We enforce this by partitioning variable declaration into two parts, as before: uninitialized declaration, then initialization. 

# Design Miscellany
Global variables are late-bound in Lox (evaluated at compile-time). Local variables realistically (that is, for the user’s sake) can’t be implemented that way, as late binding and runtime resolution is slower than early compile-time resolution, and local variables are used quite often.

The ability to resolve every local variable to a single stack slot allows us to save both time and space. We don’t need extra time at runtime to look up and resolve local variables, and we don’t need extra space to store the names of those locals for late resolution.