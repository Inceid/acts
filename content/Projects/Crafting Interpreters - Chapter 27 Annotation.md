---
tags:
  - project/annotation
  - release
date created: 2025-02-09

status: stable
---
Now we add classes and instances - this translates to a new token, struct runtime representation, and stack semantics (thes stack semantics are that a class is treated exactly like a variable. Lox allows local class declarations).

In our runtime representation, a class is essentially a variable (a name bound to a value) augmented with a hash table that maps method names to function bodies. That means that a class is initialized like a variable; its methods are accessed and assigned like variable values, and are parsed with the precedence of a function call (where the operation being parsed is the dot `.`).
