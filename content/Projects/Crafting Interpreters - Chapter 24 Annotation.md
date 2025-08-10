---
tags:
  - project/annotation
  - release
date created: 2025-01-29

status: stable
---
We implement functions.

# Function Objects 
Functions will be a [“sub-struct”](<Crafting Interpreters - Chapter 19 Annotation#Struct Inheritance>) of our `Obj` struct for objects. Functions will store their name, arity (number of arguments), and a chunk containing the code for their body.

At the moment, our compiler always assumes it is compiling code into one chunk at a given time. 

We’ll modify it so that it when it’s inside a function declaration, it will create a function object and populate that object’s chunk with the function’s body. 

To keep things uniform, when we’re *outside* a function declaration, we'll compile to a top-level function that stores code for all of the user’s top-level code. 

That means that the compiler is *always* writing to a function - it’s just that that “function” area is either the user’s top-level code environment or the body of a properly declared/defined function. The name of the top-level “function” will be `NULL` by convention; we’ll otherwise use functions’ names as their builtins or the user specifies.

# From Chunks to Functions 
Before this chapter, the VM gave an empty chunk to the compiler and asked it to fill that chunk with code. 
Now, the VM gives a chunk to the compiler and gets back a function object. The empty chunk gets filled with code and wrapped nicely inside the function object. 

# The Call Stack and Call Frames
Amazingly, local variable declarations within function scopes behave very similarly as they do for normal local variables within block scopes: in a stack-like fashion. 

The key problem, though is that we can only order local variables on a stack within the same function scope. Consider this example: 

```sml
fun first() {
	var a = 1;
	second();
	var b = 2;
	second();
}

fun second() {
	var c = 3;
	var d = 4; 
}

first();
```

The compiler can’t pin a single stack slot down for each variable, since the variables occupying slots can change between function calls:
```
line    stack state 
2       [a:1]
3       [a:1] [c:3] [dL4]
4       [a:1] [b:2]
5       [a:1] [b:2] [c:3] [d:4]
```

See how c is in stack slot index `1` after line 3, but it’s in slot `3` after line 5? There’s no one-to-one correspondence between variables and stack slots here. 

Because we can’t predict where a function’s local will be relative to locals outside of functions, we can’t have the compiler associate one stack with each variable. 

Instead, at compile time we track where a function’s locals within the same scope will be *relative to each other*. 

At runtime, we offset that by the location of the function call’s starting slot. 

The function call’s location and its subsequent reserved slots for its variables are called its “call frame”, a window of the stack it can use to stuff its variables into.

Finally, since function calls follow stack semantics (later function calls will finish first), we can create a runtime stack of callframes instead of using the heap. 
`meta:` Note that if your language supports continuations, function calls *don’t* always follow stack semantics.

# Compiler Stacks 
To finish off with function declarations that take parameters, we make one more structural change. Previously, we had one compiler struct that held bytecode for one function. But we want to be able to handle several (potentially nested) functions. 

We do this by creating a stack (implemented as a linked list stored on the native C stack) of compilers, each compiler containing data for the function declaration/definition it’s responsible for compiling.

# Stack traces 
We also add fuller debugging support by providing **stack traces** to the user when their code fails. A stack trace prints each function that was executing when a program died, as well as where in its execution it died. 

# Design Miscellany 
The key structural alteration in this chapter to enable function call semantics is shifting absolute stack indexing to relative indexing. That required many changes in the code’s architecture, chief among them:
- pushing instruction bytes onto call frames at a relative slot index rather than an absolute slot index in the VM’s original stack.

`debugging`
- When we implemented call stacks without completing the rest of the chapter, upon attempting to compile our code we got an error indicating a segfault. (I figured I would run it and try it since Nystrom gave the green light to try the code out halfway through the chapter.) This segfault went away when we completed the implementation and updated `debug.c`. I suspect this is because the code was segfaulting when attempting to print stack contents, as when we were invoking callstacks we didn’t fully update how the code 
	- Nystrom did suggest running the code halfway through the chapter, but this is probably because he wasn’t including `debug.c` as part of the compilation. Either way, the error upon entering an empty character went away when I finished the chapter. 