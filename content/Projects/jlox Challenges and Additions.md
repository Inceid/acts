---
tags:
  - project
  - achievement
  - release

status: stable
---
# Chapter 7
1. Polymorphic orderings: will think about this.
2. Added support for concat allowing either operand to be a string and the result being concatenated in `visitBinaryExpr`. 
	1. ~~If one of the values is null, we default to the other value. Hence `null + "3" -> "3"`, and `null + null -> null`.~~
		1. Actually scratch this last part - I might not want this functionality after all. Will think about it.
		2. We can just `stringify` the other arg and stick with that!

# Chapter 8
1. We added support for automatically displaying the value of an expression by modifying the following method in `Interpreter.java`:
```java
	@Override  
	public Void visitExpressionStmt(Stmt.Expression stmt) { 
    Object value = evaluate(stmt.expression); // new line
    System.out.println(stringify(value)); // new line
    return null;  
	}
```

As all statements are either expression statements, print statements, or blocks, and expression statements are the highest-order rule that contain expressions, we insert the `println` command here.

We do the same for variable declarations:
```java
	@Override  
	public Void visitVarStmt(Stmt.Var stmt) {  
	    Object value = null; // set var to null if declared 
								but not assigned  
	    if (stmt.initializer != null) {  
	        value = evaluate(stmt.initializer);  
	    }  
	    environment.define(stmt.name.lexeme, value);  
	    System.out.println(stringify(value));  // new line
	    return null;  
	}
```

2. We decided to require that variables be initialized when they are declared. The following code operationalizes this with the new `RuntimeError` when the initializer is `null`:

```java
@Override  
public Void visitVarStmt(Stmt.Var stmt) {  
    Object value = null; // set var to null if declared but not assigned  
    if (stmt.initializer != null) {  
        value = evaluate(stmt.initializer);  
    }    else {  // new line
        throw new RuntimeError(stmt.name, "Variable has not been initialized."); // new line
    }    environment.define(stmt.name.lexeme, value); 
    System.out.println(stringify(value));  
    return null;  
}
```
