---
tags:
  - project/annotation
  - release

status: stable
---
# Chapter 6: Parsing Expressions.
Chapter Themes: parsing, recursive descent

There's a massive difference between writing some regular-expression-oriented code from scratch to parse some block of text and writing a proper *parser*. Being able to do so is an impressive and nuanced skill. Onward, then.

Given a series of tokens scanned from a string, the parser must map those tokens to terminals in the grammar to figure out which rules *could have* generated that string.

This creates ambiguity. Different orderings of the same rules can account for an input string. How do we decide which ordering we want, so as to not misinterpret the user's code? Two rules help:
1. **Precedence:** a rule of priority of evaluation for a sequence of operators which are not all the same. Example: PEMDAS for arithmetic.
2. **Associativity:** a rule of priority of evaluation for a sequence of operators with are all the same. Example: if an operator is left-associative, instances on the left evaluate before those on the right. Example: `-`. On the other hand, some operators are right-associative, e.g. assignment. 

Lox will adopt the same precedence rules as C.
To implement precedence of expression types, we need to stratify them explicitly in the grammar. Why? At the moment, the two rules
`expression -> literal | unary | binary | grouping ;`
and
`binary -> expression operator expression ;`
allows any expression to be a subexpression, regardless of whether the precedence rules allow it.
Example: `3 + 4 * 5` could be interpreted as `(3 + 4) * 5` or `3 + (4 * 5)`, where we want only the second one. To explicitly require this, we modify the grammar by adding a rule for each level that matches expressions at its precedence level or higher.

Arranged from lowest to highest in precedence:
#### Precedence-Ordered Expression Grammar

```
expression    -> equality ;
equality      -> comparison ( ( "!= | "==" ) comparison )* ;
comparison    -> factor ( ( "!=" | "==" | ">=" | "<=" | ">" | "<") factor )* ;
term          -> factor ( ("-" | "+") factor )* ;
factor        -> unary ( ("*" | "/") unary )* ;
unary         -> ( "!" | "-" ) unary | primary ;
primary       -> NUMBER | STRING | "true" | "false" | "nil" | "(" expression ")" ;
```
#### Continuing...
Note that we structure all binary operator rules to match **flat sequences** of their operations rather than as recursive rules, so as to avoid having to implement left recursion (left recursion just sucks for some parsers).

The flat sequence structure looks like this for `factor`:
`factor -> unary ( ("*" | "/") unary )*`
which basically says a factor is a unary multiplied or divided by any number of additional (possibly zero) unaries.

This eliminates ambiguity up to precedence of expressions.

To actually build the parser, we use the principle of recursive descent. This principle starts from the top or outermost grammar rule and works its way down to nested subexpressions before reaching the AST's leaves, building its recursive calls along the way. Each rule corresponds to a function. The code just implements each rule of the unambiguous expression grammar into Java code.

`meta:` decided to add another helper for parsing a series of binary operators given a list of token types to handle redundancy between equality, comparison, term and factor rule implementations. See `parseBinary()` in `Parser.java`.

Parsers *must* have good error handling. This requires some hard rules:
1. Detect and report the error.
2. Avoid crashing or hanging. No infinite looping, no segfaulting.

And some basic quality-of-life guidelines:
1. Be fast. Don't keep the user waiting.
2. Report as many distinct errors as there are, altogether.
3. Minimize cascaded errors - don't report 'phantom' or 'ghost' errors that would resolve if a higher-order error were fixed.

Funny enough, a good rule for responding to an error and continuing to look for others is **panic mode**. This is where the parser notices an error, then try to align the remaining token sequence to match the current rule being parsed. This is called **synchronization**. To do this,
	1. first the parser freezes its state at some rule.
	2. It then discards tokens until it reaches one that can appear at that point in the rule. Any additional syntax errors hiding in the discarded tokens aren't reported.
Think of the above as a way of the parser saying "ah shit this one doesn't work. Let's just cut our losses and save what we can from the rest of the code". 

The parser doesn't always have to synchronize. If it does, it should unwind and synchronize (via throwing a `ParseError` object in our case - the panic button). If it doesn't it should report the error and continue attempting to parse the remaining tokens as usual.

In our case, we will discard tokens until we reach the beginning of the next statement.

We can test the interpreter by compiling Lox.java and typing in some grouped mathematical expressions with or without parentheses and seeing how Lox handles them!