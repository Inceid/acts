---
tags:
  - project
  - release
date created: 2024-10-30

status: stable
---
Initial form:
```
expression -> literal | unary | binary | grouping ;
literal    -> NUMBER | STRING | "true" | "false" | nil ;
grouping   -> "(" expression ")" ;
unary      -> ( "-" | "!" ) expression ;
binary     -> expression operator expression ;
operator   -> "==" | "!=" | "<" | "<=" | ">" | ">=" 
                   | "+" | "-" | "() | "/" ;
```

Unambiguous form (including precedence order):
```
expression    -> equality ;
equality      -> comparison ( ( "!= | "==" ) comparison )* ;
comparison    -> factor ( ( "!=" | "==" | ">=" | "<=" | ">" | "<") factor )* ;
term          -> factor ( ("-" | "+") factor )* ;
factor        -> unary ( ("*" | "/") unary )* ;
unary         -> ( "!" | "-" ) unary | primary ;
primary       -> NUMBER | STRING | "true" | "false" | "nil" | "(" expression ")" ;
```

With modifications / rules to accommodate statements, variables, and blocks:
```
program       -> declaration* EOF ;
declaration   -> varDecl 
			   | statement ;
varDecl       -> "var" IDENTIFIER ("=" expression)? ";" ;
statement     -> exprStmt
               | printStmt 
			   | block ;
block         -> "{" declaration* "}" ;
exprStmt      -> expression ";" ;
printStmt     -> "print" expression ";" ; 
expression    -> assignment ;
assignment    -> IDENTIFIER "=" assignment
			   | equality ;
equality      -> comparison ( ( "!= | "==" ) comparison )* ;
comparison    -> factor ( ( "!=" | "==" | ">=" | "<=" | ">" | "<") factor )* ;
term          -> factor ( ("-" | "+") factor )* ;
factor        -> unary ( ("*" | "/") unary )* ;
unary         -> ( "!" | "-" ) unary | primary ;
primary       -> "true" | "false" | "nil"
			   | NUMBER | STRING 
			   | "(" expression ")"
			   | IDENTIFIER ;
```

With modifications / rules to accommodate control flow:
```
statement     -> exprStmt
			   | forStmt
			   | ifStmt
			   | printStmt
			   | whileStmt
			   | block ;
forStmt       -> "for" "(" ( varDecl | exprStmt | ";" )
			     expression? ";"
				 expression? ")" statement ;
ifStmt        -> "if" "(" expression ")" statement
			   ( "else" statement )? ;
whileStmt     -> "while" "(" expression ")" statement ;
```

```
expression    -> assignment ;
assignment    -> IDENTIFIER "=" assignment
			   | logic_or ;
logic_or      -> logic_and ( "or" logic_and )* ;
logic_and     -> equality ( "and" equality )* ;
```
the precedence of the logical operators is between assignment and equality. 

With modifications to accommodate functions:
```
declaration   -> funDecl | varDecl | statement ;
funDecl       -> "fun" function ;
function      -> IDENTIFIER "(" parameters? ")" block ;
parameters    -> IDENTIFIER ( "," IDENTIFIER )* ;
...
statement     -> exprStmt | forStmt | ifStmt | printStmt |
				 returnStmt | whileStmt | block ;
returnStmt    -> "return" expression? ";" ;
...
unary         -> ( "!" | "-" ) unary | call ;
call          -> primary ( "(" arguments? ")" )* ;
arguments     -> expression ( "," expression )* ;
```

With modifications to accommodate classes and objects:
```
declaration   -> classDecl | funDecl | varDecl | statement ;
classDecl     -> "class" IDENTIFIER "{" function* "}" ;
function ...
...
assignment    -> (call ".")? IDENTIFIER = assignment |
				 logic_or ;
...
call          -> primary ( "(" arguments> ")" | "." IDENTIFIER )*
```

With modifications to accommodate inheritance:
```
classDecl     -> "class" IDENTIFIER ( "<" IDENTIFIER )? "{" function* "}" ;
primary       -> "true" | "false" | "nil" | "this" 
			   | NUMBER | STRING 
			   | "(" expression ")"
			   | IDENTIFIER 
			   | "super" "." IDENTIFIER ;

```