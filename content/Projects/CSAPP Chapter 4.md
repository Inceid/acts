---
tags:
  - project/annotation
  - pcms/cs
  - release
date created: 2025-07-18
last updated: 2025-08-07
keywords:
  - processor hardware
  - digital hardware design
todo:
  - Implement HCL descriptions for all hardware-level computations of SEQ
aliases:
  - Processor Architecture
projstatus: v0
---
# Key Points
- This chapter presents a simple instruction set “Y86-64” (inspired by x86-64) that features a select few data types, instructions, addressing modes, and a simple byte-level encoding. 
- This chapter introduces a simple language, HCL (Hardware Control Language), as a digital logic to describe the control portions of hardware systems. 
- This chapter implements SEQ, a sequential processor for Y86-64, then describes a pipelined implementation to allow a processor to effectively execute different steps of multiple instructions simultaneously. 
- The Y86-64 processor is a simplified but effective model system for studying the anatomy and physiology of modern processors.

# The Y86-64 Instruction Set Architecture
The Y86-64 ISA is defined by the following state, instructions, conventions, exceptions, and program conventions:

## State
The state consists of registers, condition codes, a program counter, memory, and execution status.

### Registers
We have 15 program registers: `%rax`, `%rcx`, `%rdx`, `%rbx`, `%rsp`, `%rbp`, `%rsi`, `%rdi`, `%r8`-`%r14`. Each stores a 64-bit word. 

Register `%rsp` is a stack pointer. Other registers have no fixed meanings or values. 

### Condition codes
Three condition codes, `ZF`, `SF`, and `OF`, store information about the effect of the most recent instruction. 

### Program counter
The program counter `PC` holds the address of the instruction currently being executed. 

### Memory
The processor memory is a large array of bytes that holds program code and data. 

### Status
A status code `Stat` indicates the overall state of program execute. It indicates either that the program executed normally or some exception has occurred. 

## Instructions 
The Y86-64 instruction set includes only 8-byte int operations, fewer addressing modes, and fewer overall operations. 

![[Pasted image 20250802115543.png]]

The above describes the Y86-64 instruction set encoding. Instruction encodings are 1-10 bytes. An instruction is encoded by a 1-byte specifier, possibly a 1-byte register, and possibly an 8-byte word. `fn` specifies an integer/movement/branch operation.

Function codes for some arithmetic, branch, and move operations are shown below. These correspond to `OPq`, `jXX`, and `cmovXX` in the previous figure.

![[Pasted image 20250802120821.png]]

Finally, here are the register identifiers for the operations that use them.

![[Pasted image 20250802121038.png]]

## Exceptions

The table below describes the possible status codes in Y86-84. The processor halts for any code other than `AOK`.

![[Pasted image 20250802122514.png]]

# Logic Design and the Hardware Control Language
In hardware design, we use digital circuits to compute functions on bits and store them. Circuits are organized around “gates”, which take in bits and return 1 (“high-voltage”) or 0 (“low voltage”) according to a set of logical rules.

Three components are required to implement digital systems via circuits: combinational logic for functions, memory elements to store bits, and clock signals to regulate updates for memory elements. 

This section decribes these components and HCL, a “hardware control language” for the control logic of a Y86-64 processor. 

## Logic Gates 
All digital circuits are organized around logic gates. 

`portal` Logic gates are a binary analog of [feedback loop elements](<Annotation of Thinking in Systems#System Structure and Behavior>).

HCL expressions for logical operations include `&&` for $\text{AND}$, `||` for $\text{OR}$, and `!` for $\text{NOT}$. 

>[!example] Example: `eq`
>Consider the function `eq` which returns whether two elements `a` and `b` have the same value.
>
>We can implement this in HCL as follows:
>	`bool eq = (a && b) || (!a && !b);`
>
>For another example, consider `xor`, implemented as follows:
>	`bool xor = (a || b) && (!a || !b)`

## Combinational Circuits 
A combinational circuit is a network of logic gates under the following restrictions:

1. Every gate must be connected to one of the following:
	1. a system input *or*
	2. the output of a memory element *or*
	3. the output of a gate.
2. Outputs of two or more gates cannot be directly connected.
3. The network cannot have a cycle.

### Multiplexers
A *multiplexer* circuit is a special kind of circuit that selects a value from a set of data signals according to a control input signal. 

In practice, multiplexers allow us to select a word from many sources according to a condition. They’re described in HCL by case expressions, which look like this:

```
[
	select1 : e1;
	select2 : e2;
	...
	selectk : ek;
]
```

where each case $i$ is defined by a Boolean expression $select_i$ indicating when this case is selected, then the result of the case $expr_i$, which is which is an integer expression.

This looks like `switch` statements in `C`, except selection expressions don’t need to be mutually exclusive. They are evaluated sequentually - the first case whose selection expression yields `1` is selected. 

This chapter implements a particularly important circuit called the *arithmetic/logic unit* or ALU, an abstract model of bit-level arithmetic that informs Y86-64’s implementation of arithmetic.

### Circuits and `C`
There are important differences between combinational circuits and `C` code:

| Circuits                                           | `C` expressions                                                                                             |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Outputs continuously change in response to inputs. | Outputs only change when encountered in program execution.                                                  |
| Arguments can only be `0` or `1`.                  | Arguments to logical expressions can be arbitrary integers.                                                 |
| Circuits don’t have any partial evaluation rules.  | Logic expressions can be partially evaluated. See [“short-circuit evaluation”](<Short-Circuit Evaluation.md>). |

## Memory and Clocking
Circuits can’t store information. They react immediately to their inputs by generating outputs. Because of this, we can’t design a processor just with circuits. We need some form of *storage* or *state*. 

We can accomplish this via *sequential circuits*, which add: 
- a *clock*: a periodic signal determining when new values are loaded;
- *random access memories*: containers for multiple words that use addresses to select words to read or write. These include virtual memory and the register file. 

### Register Files
A typical register file with its inputs/outputs is depicted below.
![[Pasted image 20250802191714.png]]

This file has two read ports A and B with addresses srcA and srcB, as well as one write port W with address dstW. 

As the register file has internal storage, it’s not a circuit. But it effectively behaves like one. 

### Random Access Memory
The Y86-64 processor implemented in this chapter will have the following overall memory model.
![[Pasted image 20250802192429.png]]

which supports: 
- a single input “address”,
- a data input “data in”,
- a data output “data out”,
- as well as read, write, error logging,
- and a clock that control when to write to memory.

# Sequential Y86-64 Implementations 
The first iteration of a sequentual Y86-64 processor is called SEQ, presented here.

## Stages
An instruction to SEQ is processed in five stages plus a final update.

### 1. Fetch
- Read an instruction from memory starting at PC. 
- Extract the instruction specifier byte into `icode` and `ifun`, possibly a register specifier byte into `rA` and `rB`, and possibly an 8-byte constant word `valC`. 
- Set `valP` to the address of the next instruction.

### 2. Decode
- Read up to two operands from the register file into values `valA` and/or `valB`.

### 3. Execute 
- The ALU executes the instruction, computes the address of a memory reference, and/or updates the stack pointer. The result is `valE`.
- Condition codes are set if applicable.

### 4. Memory 
Read or write data to memory; the value read is `valM`.

### 5. Write 
Write results to the register file.

### 6. PC update
Set PC to the address of the next instruction.

Loop indefinitely performing the stages above, stopping when an exception occurs.

### Summary
The tables below summarize how each Y86-64 instruction proceeds through these stages.

![[Pasted image 20250802194106.png]]
![[Pasted image 20250802194450.png]]
![[Pasted image 20250802194352.png]]
![[Pasted image 20250802194416.png]]

## SEQ Hardware Structure 
The following is an abstract model of SEQ’s hardware. Instruction execution begins in the lower left corner with “Fetch”. 

![[Pasted image 20250802195739.png]]

All processing occurs in a single clock cycle. A hardware unit performs each stage of processing as follows:

### Fetch
Read an instruction’s bytes from instruction memory using `PC`. Compute `valP` (which is the updated `PC`)

### Decode
Read `valA` and `valB` from the register file. 

### Execute 
Use the ALU to perform arithmetic, update the stack pointer, compute an address, or pass an input to output.

Store condition codes in the CC register. Compute new values using the ALU. 

If the instruction is a jump, compute the branch signal `Cnd` based on the condition codes and jump type. 

### Memory
Read or write a word of memory. 

### Write back
Write values computed by the ALU, or write values read from memory. 

### PC update
Select the updated PC to either be 
- `valP`, the address of the next instruction, 
- `valC`, the destination address of a call or jump, *or*
- `valM`, the return address read from memory.

A more detailed view of the hardware is shown below.
![[Pasted image 20250802234326.png]]

## SEQ Timing 
SEQ is implemented using combinational logic, clocked registers (the PC and CC registers), and random access memories (register file, instruction memory, main data memory). 

Combinational circuits don’t require sequential control - whenever inputs change, circuits spit out a modified output. The instruction memory doesn’t require extra sequential control either since it’s only used to read single instructions at a time by design.

What *does* require explicit sequential control are the PC, CC, data memory, and register file. A single clock signal controls when to load values to registers and write to memories. 

At the beginning of each clock cycle, the PC loads a new instruction. The CC register loads when an `int` operation is executed. Data memory is written only when indicated by appropriate operations.

>[!thesis] Synchronized state updates
>The key point is that, when the clock “ticks”, all register and memory values requiring updates are updated all at once. This does not violate the sequential specification of the ISA. From the machine programmer’s perspective, instructions execute one after another. But at the level of hardware, everything happens at once. Spooky. 

The corresponding principle this behavior obeys is called 
>[!thesis] No reading back 
>A processor reads back the state updated by an instruction to finish processing it. 

## SEQ Stage Implementations
HCL descriptions for the control logic blocks and detailed hardware structures to implement SEQ are shown below. 

These are HCL encodings of certain important constant values.
![[Pasted image 20250803141202.png]]

### Fetch
Zooming in on the Fetch stage of the hardware diagram:
![[Pasted image 20250803142247.png]]
To summarize the above:
- The instruction memory unit reads an instruction and splits the 0th byte into `icode` and `ifun`.  Based on `icode`, the system computes:
	- `instr_valid`: is this a legal instruction?
	- `need_regids` is there a register specifier?
	- `need_valC`: is there a constant word?
- Bytes 1-9 encode the register specifer byte and constant word. The Align unit processes these.
- The PC incrementer unit generates `valP` using the `PC`, `need_regids`, and `need_valC`.

### Decode and Write back
Now zooming in on the Decode and Write back stages:
![[Pasted image 20250803203420.png]]
These stages are combined because they both access the register file. 

The register file has four ports:
- `srcA` and `srcB` are for simultaneous reads;
- `dstE` and `dstM` are for simultaneous writes. 

Each port has an *address connection* - which is a register ID - and a *data connection* - a set of 64 wires that is either an output word or input word. 

### Execute
Diagrammed below.
![[Pasted image 20250803204834.png]]
This stage includes the ALU, which performs addition, subtraction, and, and xor on inputs `aluA` and `aluB` based on `alufun`. These three signals are each generated by a control block. The result is `valE`.

When an `OPq` instruction is passed to the ALU, a signal `set_cc` is emitted that tells the CC unit to update the condition code register. The “cond” unit uses `ifun` and the condition codes to determine if a conditional branch or data transfer has occured, and emits the `Cnd` signal to handle conditional moves and branches.

### Memory
Described below.
![[Pasted image 20250803205356.png]]
This stage reads or writes program data.

### PC Update
The new PC will either be `valC`, `valM`, or `valP` depending on the instruction type and branch decision. 

Unfortunately, SEQ is extremely slow. We can speed it up via a powerful technique called *pipelining*. 

# Pipelining 
Pipelining saves computation time separating instruction execution into stages and allowing each stage to execute independently and simultaneously with the others if it has a value to work on.

We implement pipelining in SEQ by putting clocked registers between blocks of combinational logic. 

## Limitations
Pipelining hardware systems in the real world can face major practical hurdles to its efficiency.

### Nonuniform partitioning
If some stages are much shorter than others, this creates an imbalance in the pipeline - instructions will get held up at the most time-intensive stages. 

### Diminishing returns
Pipelining only works because we are able to add pipeline registers in between stages that receive updated data at each turn of the clock cycle. Each of these registers incurs an overhead cost. 

Too many stages and you have a lot of overhead. Too much overhead and you defeat the purpose of pipelining: the cumulative cost of the pipeline registers nixes the performance benefits of pipelining. 

## Feedback and Dependency
We have to be careful to implement pipelining in a manner that respects dependencies between instructions, especially when it comes to one instruction referencing a memory location updated by a previous instruction.

# Pipelining Y86-64
SEQ+ implements a pipelined version of SEQ.

## Rearrangement
We start by rearranging the computation steps so that the PC update occurs at the beginning of each clock cycle rather than the end. 

We do this by computing the PC value for the *current* instruction. 

We then add pipeline registers between each of the stages - fetch, decode, execute, etc. - which separate each stage of instruction execution. 

The resulting processor can be called PIPE-. It’s a little worse in performance than the final product will be. 

>[!note] Note!
>Ordering pipeline staging from bottom to top allows us to preserve the convention that program flow goes top to bottom.
>
>This is because the last instruction is always at the earliest stage of the pipeline. If the last instruction should be at the bottom of a diagram, then the earliest stage should be at the bottom, the later ones at the top.

## Relabeling
Since multiple instructions are now processed at the same time, we’ll have multiple `valC`, `valE`, `srcA`, and `Stat` values for each instruction in processing. We have to be careful not to mix them together. 

For each signal stored in a pipeline register, its name is `R_name` where `R` is the register among `D` (decode), `E` (execute), `M` (memory), and `W` (write back) and `name` is the signal.

For each signal computed within a *stage* (not in a register), prefix their name with a lowercase of the stage name, e.g. `f_stat`, `m_stat`.

In general, all the information about a particular instruction should be kept in a single pipeline stage. 

## Branch prediction
Our goal with pipelining is to execute one instruction each clock cycle. Except for conditional jumps and `ret`s, we can usually determine the address of the next instruction based on the fetch stage. 
- For `call` and (unconditional) `jmp`, the location is `valC`; 
- For all others, it will be `valP`. 

Since the next instruction is usually one of two values, we can try to always load an instruction each clock cycle by attempting to *predict* the next value of the PC. For conditional jumps, we have to choose `valP` (jump was taken) or `valP` (jump not taken) and deal with the consequences of misprediction. More on that later.

In PIPE-, we assume conditional branches are always taken (the new PC is always `valC`).

We do not predict any value for the `ret` return address - it could be any location in memory. 

## Hazards 
We have to resolve the two kinds of dependencies to complete the process: 
1. data dependencies: results from one instruction are used as data in the next;
2. control dependencies: one instruction determines the location of the next one (e.g. in a jump, call, or return).

When they have the potential to cause errors, these are called *hazards* - data hazards or control hazards.

## Stalling 
One technique of avoiding hazards is to *stall* or hold back an instruction until the hazard condition has passed. Every time we hold back an instruction, we “inject a *bubble*” into the execute stage. 

A bubble is essentially a `nop` instruction - it does nothing. Unfortunately, this does come at a performance cost - registers will be clogged up with bubbles while executing hazard-heavy programs. 

## Forwarding 
If an operand to an instruction was just about to be written to a register before execution, that value can be *forwarded* to the instruction. 

This allows results from a pipeline stage to be passed to an earlier stage for a separate instruction. This requires adding extra connections and logic to the hardware. 

The book implements this by adding several forwarding connections between one stage and a previous stage, generating PIPE, an implementation of PIPE- with forwarding and hence much improved performance. 

## Load/Use Data Hazards
In some cases, forwarding won’t be enough to avoid hazards: *load/use* hazards occur when one instruction reads a value from memory while the next one uses it as an operand. 

Here, a combination of stalling and forwarding can be used. 

## Control Hazards
These occur when a processor can’t determine the address of the next instruction reliably. 

These occur in PIPE for `ret` instructions and branch mispredictions for jumps.

These are handled via careful design of the pipeline control logic and stalling where appropriate. 

- For `ret`, we automatically stall until `ret` passes through decode, execute, and memory (by injecting three *bubbles*) so that the return address can be computed from the next instruction in the fetch stage.
- For branch mispredictions, we always predict branches will be taken. If we mispredict, we can *cancel* the target of the branch by injecting bubbles, then fetch the instruction after the jump, skipping it completely.

# Exceptions 
**Exceptions** are events that break normal program execution flow. Y86-64 allows for three internally generated exceptions: (1) `halt`, (2) an invalid instruction/function code combo, and (3) an invalid memory access.

The basic idea is this: if we encounter an excepting instruction, we “pause” the system. Every instruction before the exception should be completed. Every instruction after the exception is ignored. 

How this is implemented: 
- an excepting instruction is encountered;
- the processor sets a status code, carries it forward through the rest of the processor, and reports the exception to the OS;
- based on this new status code, all subsequent instructions can no longer update the programmer-visible state. 

If multiple instructions trigger an exception, we report the one furthest along in the pipeline. This corresponds to the first instruction in program-order that triggered the exception. 

# PIPE
Below is the pipelined implementation of SEQ with performance optimizations as described above and some basic exception handling.

## PC Selection and Fetch
The fetch stage and PC selection logic of PIPE is described below.

![[Pasted image 20250810093322.png]]

The hardware units are the same as in [SEQ](<#SEQ Stage Implementations#Fetch>). 

### PC selection 
Chooses between three sources:
- `valP`, `valA`, and `valC`. 

## Execute 

## Memory 

## Special Control Cases

## Pipeline Control

## Multicycle Instructions

## Memory Interfacing 

# Lessons about Processor Design
1. Hardware resources are limited. You want to use them for optimal performance at minimal cost. The pipelined Y86-64 processor presented here accomplished this via a highly uniform framework for processing all instruction types. This allowed the same hardware units to access and manipulate the logic for all instructions.

2. The ISA wasn’t implemented directly. The Y86-64 ISA is sequential - our implementation was pipelined (parallel), but yielded the same effects as a sequential implementation.

3. Hardware design is an exact science. You have to get the chip design right on the first try - failure is very costly. This requires closely analyzing instruction and control combinations for any errors or exceptional behaviors that might arise. 