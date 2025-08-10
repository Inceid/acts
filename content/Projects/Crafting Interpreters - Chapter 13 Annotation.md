---
tags:
  - project/annotation
  - release

status: stable
---
In this chapter we implement inheritance over classes. 

Inheritance was initially implemented in Simula for developers to reuse code patterns across different parts of code. The basic idea is that, given a class `c`, you can create a new class `d` that implements and extends `c` with extra features.
In this relationship, `c` is the superclass and `d` is a subclass.

We'll use the notation `d < c` and modify the grammar as such:
```
classDecl     -> "class" IDENTIFIER ( "<" IDENTIFIER )? "{" function* "}" ;
```

Some features of our implementation of inheritance:
- In Lox, methods are also inherited. Any superclass method should be able to be called on a subclass instance. 
- If a subclass and superclass implement the same method differently, the subclass's version overrides by default. To cancel this behavior and use the superclass's version, we use the keyword `super`.

To implement `super`, we modify the grammar:
```
primary       -> "true" | "false" | "nil" | "this" 
			   | NUMBER | STRING 
			   | "(" expression ")"
			   | IDENTIFIER 
			   | "super" "." IDENTIFIER ;
```

Importantly, the `super.IDENTIFIER` implementation consists of a super *access* followed by a function call of the accessed method. That's why we use the `IDENTIFIER` notation rather than providing the argument list, to separate those two steps. Separation also allows us to access the superclass method and bind it to new variable.

We implement `super`'s lookup functionality similarly to how we implemented `this`: closures. 

This time, given a class `c` with a `super` access inside its definition, we create a new environment that binds the superclass of `c` to the `super` keyword. This environment is the closure for each of `c`'s methods.
`par:` what if we had `super1`, `super2`, and so on for inheriting from multiple disjoint superclasses? 
`meta:` note that by default this environment is chained under the global environment, unless it is defined within a local scope. 

When a method is invoked at runtime, it spawns a new environment under the superclass environment. 