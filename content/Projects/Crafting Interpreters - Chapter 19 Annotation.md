---
tags:
  - project/annotation
  - release

status: stable
---
# String Support as Obj’s
We implement support for strings. 

Strings are tricky because they can be arbitrarily long. Previously, we stored values and their “payload” - their value - on the stack in a tagged union object. This time, we need more space that can dynamically adjust based on the size of the string being allocated. For this, we use the heap. 

Whereas the data for a larger value is stored on the heap, we access that data using a pointer stored in the tagged union corresponding that value. We call any values that are stored on the heap `Obj`’s.

# Struct Inheritance 
All values stored on the heap are called `Obj`’s, but there are many different kinds of `Obj`’s. We focus on strings here, but an `Obj` could also be a class with field data or a function. How do we handle different kinds of heap payloads?

We use a technique called **struct inheritance**, which mimics inheritance in OOP. Each `Obj` starts with a tag identifying the type of object it is. Then we have the payload fields, with a struct for each type. The type-specific structs such as `ObjString`, `ObjFunction`, etc. “inherit” fields from the generic heap-allocated struct `Obj`. We keep track of which struct is being used for a piece of data by storing its type in a field called `ObjType` in the generic struct. 
`meta:` It’s actually amazing that you can perfectly emulate OOP behavior in C with struct behavior and struct inheritance!

# The one feature we care about
For Lox, we only support string concatenation, just to keep things simple. 

# And now, basic memory management
Concatenation introduces substantial potential memory leakage. To proactively track and free unused memory, we use an **intrusive list** - a master linked list with nodes that point to all the `Obj`s allocated during VM execution. This list is intrusive because its nodes are located in the `Obj` structs themselves.

`meta:` It’s an extremely bad idea to wait until late in language development to add a garbage collector. Having automated memory management integrated from early on in development ensures that (1) all references to memory are caught (as freeing behavior is implemented for the language in its most basic form, and (hopefully) extended in a principled manner), and (2) the implementer and user can write larger programs for testing without worrying about nasty hidden memory leaks. (352)

Add a garbage collector as soon as you can for any language you develop. For now, we stick with tracking memory at a barebones level: we have a linked list that stores points to all actively allocated objects over the course of a user’s program, and we free all objects at the end of a user’s program. 

# Design Miscellany 
## Struct allocation and alignment
C specifies a few rules for allocating structs in memory:
- struct fields are arranged in memory in the order they are declared;
- when structs are nested, the inner struct’s fields are expanded in-place (and not displaced through pointer abstraction);
- structs must line up exactly with their first fields in memory.
	- this implies that a pointer to a struct is equivalent to a pointer to its first field in memory - they point to the same cul de sac of memory!

## Macros 
Say you define a macro
```c
#define MACRO(expr) (condition1(expr) && condition2(expr))
```
In C, a macro is expanded by substituting the expression (*not its value*) into the macro’s body. This is bad news bears if `expr` executes side effects - you might only want the side effect to happen once, but it happens twice in this macro! 

if we defined `IS_STRING` like above:
```c
#define IS_STRING(value, type)  (IS_OBJ(value) && AS_OBJ(value)->type == type
```
and applied 
```c
IS_STRING(POP())
```
then we would pop two items off the stack, since `POP()` is substituted twice in `IS_STRING`’s body! Not good. So we wrap the body of `IS_STRING` in a function, `isObjType`. 
