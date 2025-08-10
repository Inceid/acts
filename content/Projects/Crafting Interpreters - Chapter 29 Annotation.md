---
tags:
  - project/annotation
  - release
date created: 2025-02-17

status: stable
---
Finally, we implement inheritance! With some special `clox`-specific provisions:

# Copy-down inheritance
Recall that in [jlox](<Crafting Interpreters - Chapter 13 Annotation>), we implemented inheritance using environment-chaining. Let’s say you have this scenario: 

```
class mineralocorticoid {
	origin() { print "Zona glomerulosa."; }
}

class aldosterone < mineralocorticoid {
	action() { print "activate ENaC"; }
}

var a = aldosterone();
a.origin(); // should print "Zona glomerulosa."
```

`aldosterone` inherits `origin()` from `mineralocorticoid`, its superclass. To look up the `origin()` method, `jlox` would have traversed environments back up to `mineralocorticoid`’s closure. This is nice, but inefficient. If we layered more inheritance, we’d have to jump through more environments, and do so on *every* inherited method invocation. 

In `clox`, we don’t use environment chaining. Instead, **when a subclass is created, all the superclass’s methods are copied over to it**.

This technique is called “copy-down inheritance”. It works because Lox’s classes are **closed**: when a class declaration in Lox finishes, its set of methods cannot be modified. But that might not hold for all languages - Python, for instance, does not have closed class declarations. 

# Super 
In the event that a subclass’s method and superclass’s method name collide, the `super` keyword explicitly loads the superclass’s version. 

Under the hood, `super` is a local variable reference to a class’s superclass, similar to an upvalue. That means it’s found in the scope immediately enclosing a subclass declaration. And it’s compiled exactly like a local variable. 

Per our implementation, the runtime needs three things to perform a `super` access: 
1. The subclass instance, which it loads onto the stack first; 
2. The superclass declaration, which it then loads; 
3. The name of the method to access, which we encode as the index of the method in the superclass’s constant table. 

