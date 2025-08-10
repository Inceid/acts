---
tags:
  - project/annotation
  - release

status: stable
---
Now we implement the compiler. 

Many paradigms implement compilers as having two passes over the source code. The first pass parses the code into a bare-bones data structure that encapsulates the code’s semantics (such as an AST, as in `jlox`). The second pass generates execution instructions from that data structure.

In our compiler, we use only a single pass. This works fine for Lox, which is a relatively small language and dynamically typed. But it likely won’t work for larger languages that require the compiler to have a wider field of view into the surrounding syntax of each piece of syntax. 

We’ll build the parser and code generator, then connect them with Pratt’s top-down operator precedence parsing algorithm.

# Panic Mode and Synchronization
As in `jlox`, we report errors in a manner that avoids vomiting error cascades to the user. Recall that, under the hood, if a parser finds an error at one point in a statement, it will probably find other errors afterwards. Later code usually depends on earlier code. We don’t want to report every single error that comes up - that just makes the user’s life unnecessarily miserable, especially when fixing the first error (or two) might fix the rest! 

We use panic mode again here: once an error is found, abandon the rest of the statement and skip to a synchronization point. That point will be a semicolon, which signals the end of the current statement and the beginning of the next. Until we reach that point, the parser will suppress reporting of any additional errors found. 

# From Tokens to Bytecode
We convert tokens into bytecode and stuff that bytecode into chunks as we parse the code. When one chunk gets filled, we move to the next available one.

For the code that connects token parsing to code generation, we use a table that maps each token type to a function that converts the token of that type to an expression corresponding to that token type. 

The basic behavior of these functions as implemented thus far is summarized below: 

| Expression form | Parsing function | Behavior                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| --------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| constant number | `number()`       | writes an `OP_CONSTANT` instruction and the number’s index in the constant pool to the current chunk, as well as the number’s value into the constant pool.                                                                                                                                                                                                                                                                                                                                                               |
| unary           | `unary()`        | recursively compile the operand, then write the `OP_NEGATE` instruction to the current chunk.                                                                                                                                                                                                                                                                                                                                                                                                                             |
| binary infix    | `binary()`       | assume we have already compiled the left operand. Recursively compile the right operand, then write the binary op instruction to the current chunk.<br>For `+`, `-`, `*`, `/`, Exploits the fact that each binary operator’s right operand is one precedence level higher than itself (left associativity) via a recursive call that process the right operand with `precedence + 1` precedence.<br>For ` = ` (assignment), we have right-associativity, which will evaluate the right operand with the same precedence.  |
# Declaration Cycles
The lookup table forms the core of the function base that our Pratt parser uses to parse an expression. We use a special trick to handle declaration cycles when defining this table. 

The basic problem is this: let’s say we have the following code:
```c
typedef void (*ParseFn)() // function pointer type
typedef struct {
	ParseFn prefix;
	ParseFn infix; 
	Precedence precedence; // precedence is an enum (int) type
}

static void binary() {
	operatorType = parser.previous.type;
	Rule r = &rules[operatorType];
	parsePrecedence((rule->precedence + 1));
	switch (operatorType) {
		case TOKEN_PLUS: emitByte(OP_ADD); break;
		case TOKEN_MINUS: emitByte(OP_SUBRACT); break;
		default: return;
	}
}

Rule rules[] = {
	[TOKEN_PLUS]  = {NULL, binary, 1}
	[TOKEN_MINUS] = {unary, binary, 1}
};
```
Assume `emitByte` and `unary` are defined. 

This code won’t compile. The problem is that C reads declarations top to bottom. `binary` references `rules` in its body when `rules` isn’t defined yet. But if we flipped `binary()` with `rules`, we would see that `rules` references it before it is defined. So we have a declaration cycle - there’s no way to compile this code as is. We have to introduce a helper function:

```c
static Rule* getRule(TokenType type) {
	return &rules[type];
}
```

which returns the rule at the given token as an index. We replace the lookup in line 3 of `binary()` with a call to this function. That lets us forward declare `getRule()` via invoking it in `binary()` without having to define it until later. This exploits C’s lazy evaluation of functions. 

# Pratt Parsing 
The Pratt algorithm forms the functional core of how our function table interacts with the token stream. 

Informally: we start each expression with a prefix rule, because all expressions have some kind of prefix - a parenthesis, a number, perhaps a unary operator (`!`, `-`). If no such prefix is found, we throw an error. 

Then, we expect only infix operators (if any). Given a token, we recursively parse infix operators under the following rule: as long as the next token’s precedence is higher than or equal to the current precedence, we parse it. 

Recall what precedence means for us: precedence is the numerical “rank” of an operator. The larger the number, 
- the closer the operator is to the leaves of the AST; 
- the more tightly an operator binds the operand(s) it associates with;
- the earlier the operator is evaluated in the expression it’s in.

That means that when parsing what tokens belong to which expressions, we know that an op can be parsed in the same expression group as ops with higher or equal precedence. Anytime we encounter a lower-precedence operator, we terminate the current expression group. We process the rest in a higher-level recursive call to `parsePrecedence`. Consider:

```
1 * 2 + 3
```

Which yields the following:
```
prec: 11
curr: 1    prev: 
prefix()
AST: (1)
->
prec: 8
curr: *    prev: 1
infix()
AST: (1 *)
->
prec: 11
curr: 2    prev: *
infix()
AST: (1 * 2)
->
prec: 7    *since 7 < 11, new expr group*
curr: +    prev: 2
prefix()
AST: ((1 * 2) + )
->
prec: 11
curr: 3    prev: +
infix()
AST: ((1 * 2 ) + 3)

```

Why do we do this? The key point: for an op token at a given precedence, all infix ops at the same precedence or higher belong to the same expression “group” according to the parser. This is because higher-precedence operators bind to their operands more tightly. In `2*3+5`, `*` binds `3` more tightly than `+` does. `+` is too low precedence to be included in the `(2*3)` club. 

If we thought of precedence as the height of a mountain, think of expression boundaries as valleys between mountains. Any time you encounter a valley (an op of lower precedence than before), you stop and return the expression group you have so far, then start the climb again. The next climb is a recursive call to parsePrecedence. 

Here’s the function distilled to its algorithmic core, in pseudo-python:

```python
def parse(prec, prev, curr):
	'''
	prec : (int) precedence of prev token 
	prev : (char) prev token 
	curr : (char) curr token
	'''
	prev = curr 
	curr = nextToken()
	prefix = rules[prev].prefix # look up prefix rule 
	prefix() # run it once

	while prec <= rules[curr].precedence: # recursively parse infix ops that
		prev = curr                       # don't override current precedence
		curr = nextToken()
		infix = rules[curr].infix # look up infix rule 
		infix() # run it 	
```

# Meta 
The relevant running analogy for a recursive descent parser, both in `jlox` and `clox`, is that of a hierarchy of translators that each have control or responsibility for some part of the code. 

Since the parser passes through code sequentially, only one translator can work on the code at a time. That means a translator has to do its work on the code and know when to pass control to another translator. 

Higher translators pass control to lower ones when appropriate. Lower ones finish their tasks and (unless otherwise specified) return control to their caller. 