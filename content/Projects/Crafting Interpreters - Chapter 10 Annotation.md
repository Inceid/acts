---
tags:
  - project/annotation
  - release
date created: 2024-11-24

status: stable
---
# Functions
We now implement functions.

Function calls have some special syntactic norms. First, in a function call
```
fun(x)
```
the callee `fun` can be any expression that evaluates to a function.

Second, anytime you see parentheses following an expression, that indicates a function call, like a postfix operator that starts with `(`. This operator must have higher precedence than any other.

# Local Scope
Our default implementation only chains function environments to the global environment. That means that, in the event that we write a local function `g` inside a higher-level function `f`, upon interpreting the local function and establishing its environment the interpreter will discard `f`'s environment. But if `g` references variables that are only defined in `f`'s environment, that's no bueno. So we have to store `f`'s environment to allow `g` to safely execute.

To do this, we implement **closures**, which are persistent scopes that hang onto local variables *after* the blocks instantiating those variables has executed and the interpreter has moved on. Roughly, this consists of the following:
* capture the current environment when a function is declared;
* use this environment as the parent of the function's local environment.

>Closures are quite a nifty concept when you understand them! Take for instance the code below:
```sml
var x = "owl";
fun outer() { 
	var x = "cat"; 
	fun inner() { 
		print x;
	} 
	return inner; 
} 
var closure = outer();
print "what happens when we call closure(): ";
print closure();
```
At the time that `inner()` is declared, `x` is bound to `"cat"`. Think of `inner()`'s closure as its personal backyard of variables (like an animal closure or pen). Each spot in the backyard is marked by a variable name. 
* `inner()` has a `"cat"` in its backyard, but although there's a variable with the same identifier yet different value outside its backyard, `inner()` cannot "capture" that variable. Its closure is fixed. That's **static (or lexical) scoping**.
* By this reasoning, the call `closure()` is equivalent to the call `inner()`. The value of `x` that should be printed is `"cat"`, which indeed it is. 
* Under dynamic scoping, `inner()` would get rid of `"cat"` and capture `"owl"` under `x`. 

`meta`
* When the parse reports an error on the maximum argument size being exceeded but doesn't stop reading the code, does the interpreter still catch the error and stop? What happens?
* Note the many subtle design decisions that probably make error handling a lot easier. For example, when parsing function declarations, consume the `{` of the function's body within the `function()` method *before* calling the `body()` method to parse and return the result of the body. If an error is raised, it is raised in the context of a function declaration, so you know exactly what kind of error to report rather than saying 'blocks need to have braces in front of them', which is more vague. `block()` itself was designed with this in mind too, since it already assumes `{` was checked (it doesn't check `{` itself) when it's called.
* Another design principle features here - separation of concerns, specifically structure (syntax) from behavior (runtime). We don't want to implement any inference about code *behavior* within our AST classes. So we don't represent the function's environment and declarations within the AST node - we create an implementation of our `LoxCallable.java` class, which is invoked in the interpreter at runtime. 
`portal:` [[How to Take Smart Notes - Introduction]] `:` separation of concerns = "someone else took care of it"