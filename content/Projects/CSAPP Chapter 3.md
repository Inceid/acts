---
tags:
  - project/annotation
  - pcms/cs
  - release
date created: 2025-07-03
last updated: 2025-08-02
keywords:
  - program arithmetic
  - program logic
  - control flow
  - procedures
  - arrays
  - structs
  - floating-point code
  - assembly
todo:
  - modularize
  - add examples
aliases:
  - Machine-Level Representation of Programs
projstatus: v1
---
This chapter assumes an x86-64 machine architecture unless otherwise stated. 

# Key Points 
- Computers represent human programs in *assembly code*, which they then assemble and link into executable programs. Assembly is human-readable machine code. 
- When writing in assembly code, programmers must precisely specify low-level instructions that a program uses to carry out a computation. Today, modern machines automatically generate efficient-enogh assembly code from higher-level languages. Understanding assembly nevertheless allows a programmer to understand nuances in program efficiency, security vulnerabilities, and data sharing behavior in concurrency-based settings.
- Optimizing a critical section of code often consists of compiling and manually examining the assembly code from different variations of the source code. 


>[!info] Names
A lot of the naming conventions, and occasionally some design choices in machine architecture, are due to historical events in their development. The book will mention these idiosyncracies when they arise. 

# Encodings 
## Assembly Code
The book uses the following command to compile all `C` programs for assembly analysis:
```bash
gcc -Og -o p p1.c p2.c 
```

This calls the `gcc` compiler to compile `p1.c` and `p2.c` into executable `p` with `-Og` as the optimization flag. This flag tells the compiler to create machine code that follows the same structure of the original `C` code, with minimal extra compiler transformation. 

This makes it easier to map the assembly code to the `C` code. 

The `gcc` command does a bunch of things; this process is outlined in [[CSAPP Chapter 1#compilation]].
1. The `C` preprocessor expands any \#include or \#define statements in the source code.
2. The compiler creates assembly files of each source code file with extension `.s` (e.g. `p1.s`, `p2.s`).
3. The assembler converts the assembly code into binary object-code files with extension `.o` (`p1.o`, `p2.o`).
	- Object code is machine code without addresses of global values filled in.
4. The linker merges all object-code files along with machine code for library functions, yielding the final executable `p`. 
	- Executable code is also machine code, but it’s the exact form of code executable by a processor. 
The relation between these forms of machine code and linking is described in [Chapter 7](<CSAPP Chapter 7.md>).

Computers abstract details of instruction implementations in two important ways: 
1. via an [*instruction set architecture*](<Instruction Set Architecture.md>) or ISA. The ISA defines the processor state, instruction format, and instruction effects on the state. Most ISAs assume these instructions will execute in sequence. 
	- The processor state, which isn’t visible to the `C` programmer, consists of the program counter, the integer register file, condition code registers for tracking control flow status, and vector registers that each hold `int` or `float` values.

2. via *virtual memory*, a memory model that “appears” as a large byte array. The actual implementation of memory involves a more complex interplay of hardware memories and OS software (see [Chapter 9](<CSAPP Chapter 9.md>)).
	- `C` allows different data types to be allocated in memory. But machine code sees all memory as bytes. It makes no distinction between data types. 

To generate assembly code only, we can run the following:
```bash
gcc -Og -S p1.c p2.c
```

which will generate `p1.s` and `p2.s` and go no further. 

## Disassembly
To quickly inspect the content of machine-code files, we can use *disassemblers*, which convert machine code into assembly-like format. We can do this in most linux systems via the following command: 
```bash
objdump -d p1.o
```

Some important points about machine code and “disassembly” code:
- instructions can range from 1-15 bytes. 
- from a given starting position, there is a 1-to-1 decoding of bytes into machine instructions. Ex: only the instruction `pushq %rbx` can start with byte 53. 
- the disassembler creates new assembly code based only on byte sequences in machine code. It doesn’t access the source code or previously made assembly code versions of a program. 
- the disassembler uses slightly different naming conventions for instructions, usually omitting or adding `q` after an instruction name. This doesn’t affect functionality.

To access low-level features of a machine/processor, it’s possible and sometimes desirable to integrate assembly code into `C` programs either via linkage or GCC’s *inline assembly feature* for embedding assembly into `C` code. 

Primitive data types are represented as follows in `C`. 

| `C`      | Intel data type  | Assembly suffix | Size (bytes) |
| -------- | ---------------- | --------------- | ------------ |
| `char`   | Byte             | `b` (“byte”)    | 1            |
| `short`  | Word             | `w` (“word”)    | 2            |
| `int`    | Double word      | `l` (“long”)    | 4            |
| `long`   | Quad word        | `q` (“quad”)    | 8            |
| `char*`  | Quad word        | `q`             | 8            |
| `float`  | Sngle precision  | `s` (“short”)   | 4            |
| `double` | Double precision | `l`             | 8            |
Notes:
- Pointers are 8 bytes long. 
- Assembly instructions have a suffix denoting the size of their operand, e.g. `movb` (move byte), `movw` (move word), `movl` (move double word), `movq` (move quad word). 
- different sets of instructions and registers govern `int` and `float` arithmetic, so there’s no ambiguity in `l` being used twice above. 

# Accessing Information 
Each CPU has 16 general purpose registers for storing 64-bit values, which are typically ints and pointers. Their names all bgin with `%r`.

These registers are buckets for instructions to store any temporary information into. Registers tend to serve specific roles in typical programs. 

## Operands 
Instructions typically have *operands*, which are the values to operate on and the location to store the result. 

Operands are classified into three types: *immediate* for constant values, *register* for the contents of a register, or *memory* for accessing an area in memory via an address, which denotes the starting point of the area to access. 

The most general form of a memory reference is given by the form 

$$
Imm(\texttt{r}_{b}, \texttt{r}_{i}, s)
$$

This reference has four components: 
1. An immediate offset $Imm$ 
2. A base register `r`$_b$ 
3. An index register `r`$_i$ 
4. A scaling factor $s$, where is 1, 2, 4, or 8. 

The corresponding effective address is: 

$$
Imm + \texttt{R}[\texttt{r}_{b}] + \texttt{R}[\texttt{r}_{i}] \cdot s
$$

This form is often used when referencing array elements. Other reference forms are special cases of the above. 

![[Pasted image 20250703205020.png]]

## Moving Data 
The following comprise instructions that move data between locations in registers or memory and are grouped by *instruction classes* denoted in $\text{CAPS}$, which perform the same operation over different operand sizes.

![[mov.png]]

The source $S$ is either immediate, a register address, or a memory address. The destination $D$ is either a register or memory address. A $\text{MOV}$ instruction cannot have both operands be memory addresses. 

`movabsq` extends `movq` to allow 64-bit immediate data. The regular `movq` instruction can only handle operands representable as 32-bit 2’s complement numbers, which are sign-extended to produce 64-bit values for the destination. `movabsq` can have any 64-bit immediate as its source but must have a register as its destination. 

## Pushing and Popping from the Stack 
The program stack is a fundamental data structure that stores instructions to be executed. It operates according to “last-in, first-out”: the last instruction “pushed” or stored onto the stack will be the first that’s retrieved and executed or “popped” off the stack. 

The pointer `%rsp` is the stack pointer, which holds the address of the top stack element. 

`pushq` and `popq` are the corresponding push and pop instructions. `pushq`’s operand is the source data to push, and `popq`’s operand is the destination to pop to. 

Note that programs can break this convention and access any part of the stack that they want to, as the stack shares memory with the program code and data. 

# Arithmetic and Logical Operations 
Arithmetic and logical operations are divided into four groups: load effective address, unary, binary, and shifts. 

## Load effective address 
*Load effective address* or `leaq`, a variant of `movq`, reads the effective address of the first operand $S$, which must be a memory or register address, and copies it to the destination $D$, which must be a register. Using operand forms cleverly, `leaq` can also encode addition and some forms of multiplication. 

## Unary and Binary 
Unary operations have a single operand which is a register or memory address. 

Binary operations have two operands, the second of which is used as both source and destination. The first operand can be an immediate value, a register, or memory address. The second operand can be a register or memory address. The two operands cannot both be memory addresses. 

## Shifts 
Shifts have two operands. The first is the amount to shift by. The second, the destination $D$, represents the value to shift, which can be register or memory address. The shift amount can be an immediate value or the single-byte register `%cl`. 

A shift instruction on data $w$ bits long determines the shift amount from the $m$ low-order bits of `%cl`, where $2^m = w$. Higher-order bits are ignored. 

Only right-shifting requires differentiating between unsigned and signed data. 

## Special arithmetic operations 
For operations that involve or produce 16-byte (128-bit) integers, two registers `%rdx` and `%rax` are combined to represent the result. 

Multiplication and division for 16-bit operands have one operand. The second, called the “multiplicand” or “dividand”, is assumed to be stored at `%rax`. For mult, the lower 8 bits are stored in `%rax` and the upper 8 in `%rdx`. For div, the quotient is stored at `%rax`. The remainder is stored in `%rdx`. 

# Control 
Machine code provides two ways to implement conditional behavior: 1) test data values then alter the control flow based on the result, 2) test data values then alter data flow based on the result. 

Data-dependent control flow is more common. 

## Condition codes and `set`
The CPU maintains *condition code* registers describing attributes of the most recent operation, including: 
- `CF` carry flag, a carry bit from the most recent op. 
- `ZF` zero flag: the most recent op yielded zero. 
- `SF` sign flag: the op yielded a negative value. 
- `OF` overflow flag: the op caused a two’s-complement overflow. 

Some ops set condition code values without doing anything else, e.g. $\text{CMP}$, which compares two operands. The rest are the `set` instructions.

## Jump (`jmp`)
Jump instructions allow execution to switch to a specified location anywhere in the program. Each jump instruction gets a *target* operand, which specifies the location to jump to.

Jumps can be direct (the target is encoded in the jump instruction) or indirect (the target is read from a register or memory). 

Other jump instructions are conditional - they jump to the designated target if a piece of condition code specifies to do so. 

Jump targets are most commonly encoded as the offset from the target instruction and the instruction after the jump. 

They may also be encoded via an “absolute” address, using 4 bytes to directly specify the target. 

In `C`, an if-else statement follows the template 
```
if (predicate)
	then-statement 
else
	else-statement 
```

and the assembly implementation typically adheres to this format in `C` syntax:

```
	t = predicate; 
	if (!t)
		goto false; 
	then-statement
	goto done; 
false:
	else-statement
done: 
```

Typically, conditional operations are implemented via transfer of *control*: programs follow one pathway or another depending on conditions. 

But implementing via conditional transfer of *data* is more efficient for modern processors: have the program compute both outcomes and select one based on the condition. 

>[!question] Why?
>Modern processors use **pipelining** to optimize performance. Here, an instruction is processed via a sequence of stages. This approach overlaps steps of successive instructions while performing previous instructions. 
>To do so, it helps determining the instruction sequence ahead of time. Processors employ *branch prediction logic* to try to predict which branch will execute in each conditional, and generate the sequence of instructions appropriately ahead of execution. 

Assembly implements conditional data transfer via *conditional move* instructions, which take a source register/memory address $S$ and destination register $R$. The source value is read and copied to the destination only if the condition specified by the instruction holds. 

Processors don’t need to use branch prediction for this. They simply read the source value, check the condition code, then update the destination or keep it the same. All the work for both if-else cases is done ahead of time, and one of the values is chosen before proceeding. More in [[CSAPP Chapter 4]].

Importantly, not all conditional logic can be implemented via data transfer. If one of the branches would raise an error or cause another serious system side effect, it can’t be computed ahead of time. 

In practice, GCC seems to only use conditional moves when the two expressions of an if-else can be computed easily. 

## Loops 
No loops exist in machine code - instead, conditionals and jumps are used to effectively simulate loops. 

### `do-while` 
```
do 
	body-statement
	while (condition)
```

translation: 
```
loop: 
	body-statement
	t = condition; 
	if (t) 
		goto loop;
```

### `while` 
```
while (condition)
	body-statement
```

translation “jump to middle”:
```
	goto test; 
loop: 
	body-statement
test: 
	t = condition; 
	if (t) 
		goto loop;
```

translation “guarded do”: 
```
t = condition;
if (!t)
	goto done; 
do 
	body-statement
	while (condition);
done: 
```
which expands to 
```
t = condition;
if (!t)
	goto done; 
loop: 
	body-statement
	t = condition; 
	if (t) 
		goto loop; 
done: 
```

### `for`
```
for (init-expr; condition; update-expr)
	body-statement
```

The `C` standard states that this loop is equivalent to the following `while` loop code:
```
init-expr;
while (condition) {
	body-statement
	update-expr;
}
```
translation: same as `while` loop translation.

Hence all forms of loops admit of transformations into conditional jumps.

### `switch`
A `switch` statement provides multiway branching based on an integer index for cases with many possible outcomes. 

`switch` instructions use a *jump table*, an array where entry *i* is the address of code to execute when the switch index is *i*.

The time taken to perform a switch is independent of the number of switch cases, as any given case is accessed and executed via a jump table index - this is not the case for if-else statements. 

# Procedures 
A **procedure** is a segment of code with a *name*, an input set of *arguments*, and an optional *return value*. 

Good procedures: 
- can be invoked from different points in a program; 
- hide detailed implementation of an action; 
- peovide a clear and concise interface deinition of its function and effects on program state. 

Procedures appear as functions, methods, subroutines, handlers, and other kinds of code packages in different languages. 

Machine-level support for procedures requires handling several key attributes: 
- control passing - control must be passed in an organized manner from one procedure to another and back. 
- data passing - procedures must be able to pass parameters and return values to each other. 
- memory - procedures must be able to allocate and deallocate memory as needed. 

## The runtime stack 
Procedures in most languages use last-in first-out management as provided by a stack. The stack and program registers store all information required for passing control/data and allocating memory. 

Space on the stack can be allocated by decrementing the stack pointer by the appropriate amount; space can be deallocated by incrementing the pointer. 

This is useful for procedures, which allocate stack space when registers don’t have enough space for their data. This stack space is called the procedure’s **stack frame**. 

## Control passing 
Say procedure `P` calls `Q`. At the point it calls `Q`, we can set the program counter (PC) to the starting address of `Q`’s code. But we have to remember where in `P` we were before we descended into `Q` and return back to it. 

So before setting PC, we push the address of the spot in `P` where we called `Q` onto the stack. This is called the *return address*, which is the address we return to after the call to `Q` is finished. 

## Data Transfer 
Procedure calls may involve passing in and returning data/values. Most of this data passing occurs via registers. 

So when `P` calls `Q`, the code for `P` first copies the arguments for `Q` into the proper registers. Then when `Q` returns to `P`, the code for `P` can access the returned value in `%rax`. 

## Storage in registers 
Registers `%rbx`, `%rbp`, and `%r12-%r15` are *callee-saved registers*. When `P` calls `Q`, `Q` must preserve the values of these registers. It can do so by either not touching them, or pushing their original value onto the stack, modifying arbitrarily, then popping the original value off. 

All other registers (except for the stack pointer `%rsp`) are *caller-saved* registers. They can be modified by any function. If `P` wants any of this data preserved, it takes responsibility to save the data somewhere else before calling `Q`.

## Recursive Procedures 
Procedures can call themselves recursively. Each call has its own private stack space, so local variables of multiple calls don’t interfere with one another. 

Moreover, the stack discipline of memory allocation-deallocation naturally matches the call-return ordering of functions. 

# Arrays 
Arrays aggregate scalar data into larger data types. 

For data type $T$ and integer $N$, we declare an array $\texttt{A}$:
$T \quad \texttt{A[}N \texttt{]};$

with starting location $x_{\texttt{A}}$.

This declaration:
- allocates $L \cdot N$ contiguous bytes in memory, where $L$ is the size of data type $T$.
- creates an pointer $\texttt{A}$ to the beginning of the array. Its value when dereferenced is $x_{\texttt{A}}$.

Element $i$ is stored at address $x_{\texttt{A}} + L \cdot i$.

## Pointer Arithmetic 
If `p` is a pointer to data of type $T$ and its value is $x_\texttt{p}$, then `p+i` has value $x_\texttt{p} + L \cdot i$, where $L$ is the size of data type $T$. 

>[!tip] Takeaway
>pointer values are scalable by the size of their data type. So `p+i` scales `p` by `i` times the size of `p`’s data type.

## Nesting 
Arrays can nest inside arrays and follow the same indexing convention. Array elements are ordered in memory (and hence indexed) in *row-major*: all elements of row 0 are first, then elements of row 1, and so on. 

For an array with declaration 

$T \quad \texttt{D}[R][C]$

the element `D[i][j]` is at address 

$$
\& \texttt{D[i][j]} = x_{\texttt{D}} + L(C \cdot i + j)
$$

we traverse $C \cdot i$ entries over to get to the beginning of row $i$, then index into the $j$’th offset in that row. 

## Fixed & Variable Sizing 
`C` allows for fixed and variable sizing of arrays. 

Varaible-sized arrays can be declared as such: 

$$
\texttt{int A[}e_{1}\texttt{]}\texttt{[}e_{2}\texttt{]} 
$$

where $e_{1}$ and $e_{2}$ are evaluated at time of declaration to determine array size. 

# Heterogeneous Data Structures 
To combine objects of different types, `C` implements `struct`s or structures, which aggregate objects into a single unit, and `union`s, which allow an object to be referenced via several different types. 

## `struct`
A `struct` creates a data type that groups objects of (possibly) different types into a single object. Its components are referenced by names.

All components are stored contiguously in memory. A structure is referred to by a pointer to the address of its first byte. 

The compiler maintains information about structure components via an offset from the start byte that indicates the start of the component. 

Consider this example:

```
struct rec {
  int i; 
  int j; 
  int a[2];
  int *p;
};
```

![[Pasted image 20250713135826.png]]

The compiler maintains offsets at 4, 8, and 16 to allow access to each field, and applies the appropriate offset in the code it generates to access the field. 

## `union`
Unions allow us to circumvent `C`’s type system, allowing a single object to be referenced by multiple types. 

The key difference from `struct`s is that all fields reference the same exact block of memory. The max size of a union is the size of its largest field. 

`union`s are implemented with tags, which are enumerated types specifying the possible choices for the union. 

## Data Alignment 
Computer systems typically restrict allowable addresses for primitive data types to be a multiple of some even value, usually 2, 4, or 8. 

This simplifies work for the processor. If a machine’s software can guarantee that a processor always uses an address of $8k$ to fetch 8 bytes from memory, then it only needs one operation to find its data. Otherwise, the data might be split across two 8-byte blocks and require two access ops. 

x86-64 places no absolute restrictions. But Intel recommends that data be aligned whenever possible to improve memory performance. This begets the:

>[!rule] Alignment Rule
>Any primitive object of $K$ bytes must have an address that is a multiple of $K$. 

The compiler places directives in its assembly code indicating desired alignment for global data and sorts data accordingly. 

For `struct`s, the compiler **pads** fields with gaps that don’t fit its alignment directive so that the overall `struct` satisfies it. If needed, it also pads the end of the `struct` to fit alignment. 

# Combining Data and Control
## Pointers
A **pointer** is a reference to some location that holds data. 

Basic rules:
- Every pointer has a type. If an object has type $t$, the pointer has typr $*t$. 
- Every pointer has a value. The value is an address of an object in memory.
- Pointers are created with the `&` operator.
- Pointers are dereferenced with the `*` operator.
- Array names can be referenced (but not updated) as if they were pointer variables.
- Casting from one pointer type to another changes its type, but not its value. 

## The GDB Debugger 
`gdb` is a debugger that allows the programmer to study their code during execution. 

The programmer first sets breakpoints near points of interest in their code. When `gdb` runs, it runs the program and pauses at each breakpoint. At each breakpoint, we can look at the values stored in different registers and memory. We can “step-through” the program, running a single instruction at a time, to see what happens. 

## Out-of-Bounds and Buffer Overflow 
Because `C` does not perform bounds checking for arrays, writing to an out-of-bounds element can cause serious errors.

One example of such an error is **buffer overflow**. This is when a function that allocates to memory on the stack overflows its designated area and overwrites other program state, causing errors. 

A more malicious version of buffer overflow involves feeding a program a string that is a byte encoding of some executable code, called *exploit code* followed by bytes that overwrite the return address with a pointer to the executable code. The `ret` instruction then instead jumps to the exploit code. 

Modern compilers, especially recent versions of `GCC` for Linux, have implemented some fundamental mechanisms to mitigate these attacks.
### Randomization
Stack randomization varies the position (address) of the stack from one run of a program to another. 

### Stack Corruption Detection
Recent versions of `GCC` store a canary value randomly generated at runtime in the stack frame between each local buffer and the remaining stack state. If this canary value is altered, the program aborts with an error before returning. 

### Limiting Executable Code 
Machines also typically restrict what areas of memory can be written to or hold executable code.

# Floating Point Code 
**Floating point code** consists of processor code that governs how floats are stored and accessed, instructions for operating on floats, conventions for passing floats as arguments to functions and returning them, and for preservation of floats during function calls. 

x86-64 machines use *media registers* to hold floating-point data. Registers are called “XMM” or “YMM”, holding 128 and 256 bits respectively (16 and 32 bytes). 

Current machines use an “AVX architecture”, which provides 16 YMM registers named `%ymm0-%ymm15`, each 32 bytes long. Assembly code refers to the registers as `%xmm0-%xmm15`, where each XMM register is the low-order 16 bytes of the corresponding YMM register. 

## Movement and Conversion 
![[Pasted image 20250802093752.png]]

`GCC` uses the instructions above to shunt floating point data from memory to XMM registers, XMM registers to memory, and between XMM registers.


![[Pasted image 20250802095100.png]]
![[Pasted image 20250802095114.png]]

The instructions above convert between floats and ints, and between different float formats. 

When converting floats to integers, these operations *truncate* - they round floats towards zero.

## Procedures 
XMM registers are used for passing floating point arguments to functions and returning floating point values from them according to the following conventions:
1. there are eight registers to hold floats: `%xmm0-%xmm7`.
2. a function that returns a float passes it to register `%xmm0`.
3. All XMM registers can be overwritten by the callee. 

## Arithmetic 
![[Pasted image 20250802101924.png]]

Floating point arithmetic operators have 1-2 source operands and a destination operand. 

The first source $S_1$ can be an XMM register or a memory location. The second source $S_2$ and the destination $D$ must be XMM registers.

## Constants 
Float arithmetic is largely the same as for ints, except that immediate values - constants - cannot be supplied as operands.

The compiler must allocate and initialize storage for constants; the code reads constant floating-point values from memory.

## Bitwise operations 
Some bitwise operators can be applied as convenient floating-point manipulations. 

## Comparison 
![[Pasted image 20250802103243.png]]

The above instructions effectively implement $\text{CMP}$ for floats. 

Condition codes for the results are set as follows:

![[Pasted image 20250802103636.png]]

The unordered case occurs when $S_2$ or $S_1$ are $NaN$. 
