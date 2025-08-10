---
tags:
  - project/annotation
  - release

status: stable
---
# Tagged Unions
In this chapter, we implement value types - `nil`, `Boolean`s, and others. 

When choosing a value representation, we have to answer two questions.
1. How do we represent the type of a value?
2. How do we store distinct values?

We implement this with **tagged unions**: briefly, each value contains a type “tag” and payload for the value.

**Unions** are a data structure in C that are like structs - they allocate a block of memory for categorical data. But unlike a struct, all of a union’s fields overlap in memory. Each field in a struct has its own memory enclosure - think suburbs. In a union, all property is shared - think commune. 

The size of a union is the size of its longest/largest field. The C compiler also adds padding in between a tag (4 bits) and its union (8 bits) to enforce 8 bit boundaries in a (standard) 64 bit architecture.
- `meta:` Notice that this mimics the “lifetime” of an element on a stack. The size or height of a stack is the height of the “longest” distance between an operator and right operand. 
- `meta:` Notice how we organize aligned macros in `value.h`. 
```
#define BOOL_VAL(value)   ((Value){VAL_BOOL, {.boolean = value}})
#define NIL_VAL           ((Value){VAL_NIL, {.number = 0}})
#define NUMBER_VAL(value) ((Value){VAL_NUMBER, {.number = value}})
```

The size of a macro’s line is determined by the size of the longest instruction, with spaces inserted as padding in between, just like how the C compiler inserts padding to enforce boundaries. 

# Runtime errors
We report runtime errors by looking up debug information in the chunk at the current instruction pointer index *minus 1*, as the compiler advances past an instruction *before* executing it. So if an instruction executed and failed, it’s the one that we just advanced past. 

# False, True, Nil 
Now we get into our groove of adding/extending representations to the compiler: we move along the execution pipeline from `chunk.h`/`value.h` → `scanner.c` → `compiler.c` (parser) → `vm.c` (interpreter) → `debug.c` (disassembler). The scanner already has functionality for reading false, true, and nil keywords, so we update the parser by adding appropriate functions into our rules table and update the interpreter by pushing the appropriate literal values onto the stack.

The approximate workflow:
* Case 1. We have a new data type we want to add. 
	1. We update `value.h` to add Lox runtime representation support for that data type;
		- If needed, we add macros for simple manipulations involving that data type.
	2. We update `scanner.c` to add support for scanning tokens representing that data;
	3. We update `compiler.c` to add parsing functions and storage support for emitting that data into chunks;
	4. We update `vm.c` to add interpreter support for executing instructions on that data and reporting any errors encountered during execution.
* Case 2. We have a new operator type we want to add for a new data type. 
	1. We update `chunk.h` to admit new OpCode for that operator; 
	2. We update `compiler.c` to add rules for converting tokens into parsing functions for that operator/operand type. 
	3. We update `vm.c` as above to add interpreter support for pushing the OpCode onto the stack and executing it.

# Equality and Comparison
For instructive purposes, we implement ` == `, `>=`, `<=`, `>`, `<`, and `!=` using just `<`, `>`, ` == ` and negation. It would technically be better for performance purposes if we had six primitive operators rather than three primitive and three higher-order forms.

An important design consideration when we compare values under the hood in C: we *cannot* call `memcmp` to compare values by directly comparing memory bit by bit. This is because tagged unions have padding bits. We have no idea what C sets those bits to, but they contain something, and might hence violate bit-by-bit equality comparison. 

So we implement equality by checking the `type` and `as` union fields for the appropriate type. For Lox code `true == false`, we check `type = BOOL_VAL` for both values, and `as.boolean(true) == as.boolean(false)`.

