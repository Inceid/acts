---
tags:
  - project/annotation
  - release
date created: 2024-11-30

status: stable
---
In this chapter we implement classes and methods (object-oriented programming or OOP). 

OOP is generally useful when you want to bundle data together with the code that operates on it. 

# The Basics
Terminology:
* The blueprint for the bundle is called a `class`, more specifically a `constructor` for that class. 
* The data is stored in the `fields` of that class. 
* The code that operates on that data is stored in the `methods` for that class. Methods can modify field values and define new fields and their values.
* A specific instance for that class with all the data filled in is called an `instance` of the class. 

We follow the same technique as before by defining a new AST node for a class declaration, which includes the name (token) of the class followed by a body enclosed by brackets of method (function) declarations. 
# Instances
For instances, we will follow a minimal approach: we already implemented function calls, so we will implement instantiations of class objects using function call syntax.

When implementing *properties* on instances, we follow Python's lead: every instance is an open collection of named values. It is "open" in the sense that any code outside of the instance's methods can also modify those values.

The grammar rule that implements this deserves some consideration:
```
call -> primary ( "(" arguments> ")" | "." IDENTIFIER )*
```
Whereas when we implemented functions `call` evaluated to a primary expression followed by a list of arguments, the rule that adds support for properties consists of a primary expression (an identifier or evaluable expression) followed by an optional mixed list of arguments and property references. This formula supports, for instance, `grandchildren(hannah, grace, chris).ages.toYear()`.

# Methods 
For the most part, we handle methods like function calls. A sticky issue arises, however, when we separate methods from their object identifiers. We want methods to be able to be bound as first-class functions for free use. But it's unclear whether methods should behave *only* like functions, in a manner unbound to their class, or whether their behavior should be *bound* to their class instance. 

Following Python and C#, we implement the second option, making methods **bound methods**. For example, given code
```sml
class Person {
	sayName() {
		print this.name;
	}
}
var jane = Person();
jane.name = "Jane";
var bill = Person();
bill.name = "Bill";

bill.sayName = jane.sayName;
bill.sayName();
```
we will have methods bind `this` to the original instance when the method is first grabbed. So when `bill.sayName` binds `jane.sayName`, any invocation of `bill.sayName()` after the method reassignment *should* return `"Jane"`, since `jane.sayName` belongs to the instance `jane` and hence *should* return `jane`'s information. 
# This 
We have implemented fields and methods, but for methods to be able to access fields of their associated instance (as well as other methods in that instance), we need a notion of `self` for methods and field to associate their identity under. 
We will follow Java-esque syntax and use `this` in place of the `self` keyword.

Critically, a method needs to "remember" the instance (the `this`) after it is accessed, even when it is bound to some other variable, so that it can refer to that instance's data whenever it is called. And it has to remember the instance at the time it is accessed.
	This is exactly what closures do for local functions! And we already have the syntax for that. Woo!
	In reality, we handle `this` with methods almost exactly like we handle closure variables with local function declarations. 

# Constructors 
Finally, we add **constructors**, which are syntactic structures that allocate memory for a new class instance and initialize its state. We have already implicitly implemented the former by adding allocation behavior for creating new instances.

To implement user-defined initialization of those instances, we'll follow Python by adding support for an `init()` method.

A special case: in an effort to keep jlox compatible with clox, we allow subsequent explicit calls to `init()` as a method to return `this` in this current for. Hence the following line in `LoxFunction.class`:
```java
if (isInitializer) return closure.getAt(0, "this");
```

`meta:` Design notes 
* Note again that, like in functions, separating declaration and definition (binding a value to a variable) allows us to reference names of classes inside their own methods. If we coupled declaration to definition, the interpreter would demand a concrete definition of a class every time it was declared. But then it would demand that of each invocation of the class inside its methods. And that would generate infinitely recursive complaints of the same problem.

`meta:` Adding `quit()`
- [[12.15]]: In the style of python, and in anticipation of this being either something Nystrom recommends I implement as a challenge or implements himself, I'll add `quit()` as an extension class of `RuntimeException` just like `Return`.
- [[12.16]]: In hindsight, implementing `quit()` as a runtime exception doesn't really work, because by construction runtime exceptions maintain the interpreter's ability to ask the user for additional input. Per brief discussion with GPT, it's better to add it as a native function (rather as a global function in the JVM) that prints an error message and calls `System.exit(0)`, as such
```java
globals.define("quit", new LoxCallable() {  
    @Override  
    public int arity() { return 0; }    

	@Override  
    public Object call(Interpreter interpreter,  
                       List<Object> arguments) {  
        System.out.println("Exiting... \n");  
        System.exit(0);  
        return null;  
    }
```

`par:` Questions
1. What's the difference between a field and a property again?
