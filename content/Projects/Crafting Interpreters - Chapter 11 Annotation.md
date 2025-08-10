---
tags:
  - project/annotation
  - release

status: stable
---
In this chapter, we resolve some residual issues regarding dynamic scoping that snuck into our interpreter when we implemented closures last time.

Consider the following:
```sml
var a = "global";
{
  fun showA() {
    print a;
  }
  showA();
  var a = "block";
  showA();
}
```
A few key points:
* This program never reassigns variables. The outer global `a` is a different variable than the inner local `a`.
* We intend Lox to be statically scoped. `showA()`'s closure is statically fixed to map `a` to `"global"`, as that was its assignment at the time of declaration. 
* So `showA()` should print "global" both times. But it doesn't. 

What happens when this program executes? It prints 
```
global
block
```
which is incorrect.

The reason it does this has to do with how we handle variable bindings within blocks. Specifically, we assumed that all the code within a block was in the same scope. In other words, given code that looks like this:
```
var x = "part of the closure";
{
	fun somefunction() {
		print x;
	}
	somefunction()
	var x = "not part of the closure"
	somefunction()
}
```
the closure passed into `somefunction()` would be a *reference* to the environment at the time of declaration. But environments are mutable objects! As the new `x` is added to the environment, the closure is also mistakenly updated. This is wrong: closures should not change. They are frozen snapshots taken *exactly* at the moment of function declaration.

To fix this, we need to change our interpreter so that for each variable expression, it tracks down the declaration for that variable to the same declaration before runtime, and does not change what this declaration is.

The tracking-down is called **resolving** variable bindings. Doing this tracking all ahead of runtime is called **semantic analysis.**

The technique we use to make sure a variable usage always resolves to the same declaration is that we: 
1. count the number of "hops" away the declaration is from the variable's usage up the environment chain (this is the **scope distance**),
2. hop that number of times up the chain to find the declaration!
	* this has the crucial advantage of allowing us to skip erroneous declarations like in the example above. 
	* Because we know we want to hop all the way out to the declaration where `x` maps to `"in the closure"`, we know that the second call to `somefunction()` has to hop **two** environments out, ignoring the middle environment where `x` is bound to `"not in the closure"`.

After the Parser creates the complete AST of the code, but before the Interpreter executes the code, we do a single pass over the AST to resolve variables in this way. The code that does this is in the `Resolver` class.

The `Resolver` class is essentially a visitor for all AST nodes that stops to resolve any variable invocations it sees. Variables are invoked or scoped in one of four cases:
* in a block statement,
* in a function declaration,
* in a variable declaration,
* in variable and assignment expressions. 

A few details:
* The Resolver also nests lexical scopes like the Interpreter. But whereas the Interpreter uses linked lists of environments, the Resolver uses a stack of `Maps`. That means that resolving a variable means adding it as an entry to the innermost `Map` on the stack.
* Per our design, if an initializer statement circularly refers to the variable being initialized, we report an error. 

Once we finish our Resolver and call it within the interpreter, we will store the resolution information (specifically the scope distance) in a map that maps an AST node with its resolved data ()

`par:` [[Syngenesis]] technique: prioritization
When you're working through a text, especially a text with unfamiliar concepts on multiple dimensions like Crafting Interpreters (such as Java code, software architecture concepts, programming language concepts), it's important to aggressively prioritize which concepts take precedence for the main project and allocate your time appropriately according to that precedence. Writing the `jlox` compiler and getting to `clox` is more important than anything else. Secondarily, we want to understand the major concepts behind programming language design and implementation. We also want to learn about how to work through a book this meaty over a long period of time without losing momentum for too long. Details about Java syntax and design idiosyncracies/details come close to last in this list. This allows us to discard some concepts about Java syntax and design idiosyncracies that evade our understanding at the moment, since we're focused on high-yield concepts only for the time being. 

`meta:` Design notes 
* For handling function declarations, in the interpreter, we do *not* visit the function's body. We do that in function calls. In the resolver, we *do* visit the function body and operate on it (resolving any variable assignments/declarations within it). This speaks to the difference between resolution and interpretation: resolution is a static analysis, hence all code that must obey static rules is checked (all branches of a conditional, the components of a function's body, etc.). Interpretation is a dynamic analysis. 
* The Resolver and Interpreter classes are tightly coupled: the Interpreter can and will throw mysterior errors and crashes if the Resolver hasn't done its job properly, precisely because the Interpreter has delegated a key static analysis to the Resolver and needs that analysis to be airtight in order to itself function safely. This coupling is a frequent source of bugs, so it's important to check that the correspondences between these two classes assures that all cases are correctly taken care of.
* Possible additional analyses that may be useful:
	* warn the user if there are local variables declared but never used (kinda like what IntelliJ does!);
	* warn the user if there is unreachable code after a return statement (that the user may want to execute given the structure of the code).