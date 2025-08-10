---
tags:
  - project/annotation
  - release
date created: 2025-02-01

status: stable
---
Now we implement closures via a two-part approach: (1) for locals that are not captured into closures, we use stack semantics as before; (2) for locals that are captured into closures, we lift those locals onto the heap where they can live as long as they are needed. 

Again, recall that a closure is a persistent “enclosure” of defined variables for general use. 

# Closure Obj’s 
Previously, we implemented runtime representation for functions in `ObjFunction`. This stored the name, arity, and code for the function. This maps one-to-one to the compile-time representation for a function, as a name, parameter list, and body is all that we have at compile-time. 

Since we already implemented local variables, this compile-time-oriented representation was enough to run functions. But this is not enough to accommodate closures. 

For closures, we wrap `ObjFunction` inside an object `ObjClosure` that takes a snapshot of the environment surrounding a function declaration at the time that it is *called* (not at the time it’s declared). 

After every function declaration, we wrap the new `ObjFunction` into a `ObjClosure` object. Now the VM only handles `ObjClosure`s rather than `ObjFunction`s. 
- Why do all this when most functions might not even need to be put into closures? Again, the idea is simplicity through uniformity here. We might pay an extra cost in memory for storing `ObjClosure` header space with the raw function code, but doing so keeps the compiler uniform in how it handles different kinds of functions.

# Implementation: Upvalues 
## Description
There’s a problem, though. Currently, we can only read and write locals for a function within its specific stack window (callframe). 

But to read a closed-over local, we need to exit the current callframe and look backwards in the code. We have no way of doing that. 

We could shunt locals from the stack to the heap as persistent data. But that would require a second pass, and we’re designing `clox` as a single-pass compiler. 

The remaining solution, then, is to load locals onto the stack, then, when they are closed over, declare an array of “**upvalues**”, which are pointers to the locals included in a closure. 

The point here is that closures are instantiated *without changing how we compiled prior code*. A new array spawns to store any locals that are defined in the environment surrounding a function declaration. (`meta` Damn what a sentence) 

Upvalues serve as a required layer of indirection to locate a captured local even after it exits the stack. 

## Implementation
Previously, when resolving locals we only looked at a function’s local scope then defaulted to the global scope for the sought-after variable. Now, we add steps to search the local scopes between a global scope and local function scope.

We also store the stack slot index of each local variable closed over. 

One important implementation detail: closures capture *variables*, not values. What that means is that when a closrue captures a variable and that variable’s value shifts at the scope it is defined, it shifts for the closure as well. 

Generalizing: all closures that capture a variable retain a reference to its location. If the value at the location changes, it changes for all closures seeing it.

Finally, for each closure that closes over a set of locals, we want to maintain references to those locals after the function declaration they are captured in. We do this by “closing” upvalues that persist when a function declaration ends - in other words, we hoist them onto the heap by loading each of them into an `ObjUpvalue` object. 

In practice, this looks like ending a function declaration, then starting from the beginning of a function’s callstack window on the stack, copying upvalues into their own structs in a `closed` field for persistent use. 

