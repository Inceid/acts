---
tags:
  - project/annotation
  - release
date created: 2025-02-10

status: stable
---
Now we implement methods and initializers. 

From prior chapters, we already have the machinery necessary to implement methods. Classes store a hash table of methods. Each key in the hash table is a method name; each value is a closure representing a method’s body. 

# Method References 
One snag in implementing methods is that *method accesses are separated from method calls* in Lox. That means 

```
var closure = instance.method;
closure();
```

is a valid piece of code. So we have to implement accesses and calls for methods separately. This entails that, just like in closures, a method needs to remember the instance that the method was accessed from. Recall that this instance is called the **receiver** of the method call. 

> We keep track of the receiver so that we can use it inside the methods body via the `this` keyword.

We implement this with **Bound Methods**, which are methods augmented with their corresponding accessing instance. Because methods are essentially functions that belong to an instance, and because bound methods are methods with their paired instance, we can extend our existing implementations for functions/closures to account for method invocations. 

# This 
To implement local references to the receiver inside a method, we use the same basic idea as in [`jlox`](<Crafting Interpreters - Chapter 12 Annotation#This>). We compile `this` as a local variable that is automatically initialized to the receiver instance and cannot be assigned to. 

This allows `this` to follow local variable semantics, which is very useful when nesting functions inside methods. Specifically, when you declare a function inside a method, and the inner function uses `this` in its body, `this` will correctly resolve to the receiver all the way on the outer scope. Since methods behave like closures, when they’re compiled, `this` will be captured and stored in a upvalue for later use.

In terms of where the instance that `this` refers to lives in memory, we use stack slot zero of the method’s callframe. 

# Initializers 
Instance `init` functions or initializers are a special kind of method in three ways: 
1. They’re automatically called when a new instance is created; 
2. `init` always implicitly returns the initialized instance and hence cannot return anything explicitly.

We add `init` support for instances by extending methods with these extra constraints.

# Design Miscellany 
- Fields take priority over and shadow methods. 
- Interesting note about OO design: tying state with behavior necessitates that an object’s state can *only* be modified by its methods. In a way, this gives OOP languages more security than imperative languages without objects: as long as your methods satisfy certain guarantees or contracts, they can make sure the data doesn’t get surreptitiously modified or destroyed when they run. 