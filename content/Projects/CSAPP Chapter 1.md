---
tags:
  - project/annotation
  - pcms/cs
  - release
date created: 2025-06-21
last updated: 2025-06-22
aliases:
  - A Tour of Computer Systems
projstatus: v1
---
# intro
A **computer system** is a set of hardware components and software that works together to run programs. 

All systems have similar hardware and software that perform similar functions. And understanding how these components work makes you a much stronger programmer. 

This book begins by completely tracing the lifetime of the `hello` C program from the moment of its creation to its execution and termination. 

```c
#include <stdio.h>

int main() 
{
	printf("hello, world\n");
	return 0;
}
```

# source code
The program begins life as a **source program** that the programmer saves as `hello.c`. This program is a sequence of 0/1 bits organized into 8-bit chunks called **bytes**. Each byte is one character in the program.

Computers represent each character with a unique byte-sized integer per the ASCII standard. 

This gets us to our first concept: 
>[!thesis] Everything is Bits
>All information in a computer is a bunch of bits - binary digits. 

What distinguishes different data objects is their context - the same sequence of bytes can represent an int, a float, a character, or instruction in different contexts. 

# compilation
`hello.c` is converted into machine-executable instructions by a **compilation system**, which consists of four programs: a preprocessor, compiler, assembler, and linker. 
- The *preprocessor* reads `hello.c` file and modifies it according to its `#include` statements, inserting code from `stdio.h` into the program text, yielding `hello.i`, another C program.
- The *compiler* translates `hello.i` into a text file `hello.s` written in *assembly language*, which describes low-level machine language instructions in textual form. Assembly is a universal interface between human-readable languages and machines. 
- The *assembler* (`as`) translates `hello.s` into a *relocatable object* file `hello.o` consisting of machine-language instructions in binary bytes. 
* The *linker* (`ld`) merges `hello.o` with `printf.o`, an auxiliary object file part of the standard C library. 
The result is the `hello` file, a fully compiled version of the `hello.c` program which can be run by typing `./hello`.

# this matters 
Understanding the underlying architecture of computer systems is not strictly required for programmers to produce good code. 

But most large, efficient software systems are designed in C. And for C programs to be designed efficiently, a programmer needs to understand when to use different kinds of C statements (e.g. `switch` versus `if-else`, `while` versus `for` loops, pointers versus array indices). Small design considerations can make big differences in program efficiency, security, and correctness. Similarly, machines use a fairly specific hierarchy for organizing memory. For a C programmer to use a computer’s memory optimally, s/he needs to understand its underlying organization. 

In both cases, making the right choices requires some fundamental knowledge of system design.

# execution and hardware 
At this point, we run `./hello` in the Linux *shell*, which is a command-line interpreter that asks for a user’s input, then performs the command given.

To understand what happens to our program when we run it, we have to understand a bit about hardware. 

## buses 
buses are electrical conduits that carry fixed-size chunks of bytes of info, called *words*, between system components.

## i/o devices
I/O devices connect a system to the outside world. These are things like mice, keyboards, display screens, and disk drives for long-term storage. 

Each I/O device is connected to the I/O bus by either a *controller* or *adapter*, which transfers info from the I/O bus to the I/O device. 

## main memory 
This is temporary storage for a program and the data it manipulates during runtime. Physically, main memory consists of **dynamic random access memory** chips. Logically, it’s a zero-indexed linear array of bytes. As mentioned before, bytes encode data and instructions.

## processor 
The **CPU** executes instructions stored in main memory using a *register* (storage device for words) called the *program counter* (PC). The PC always points at some machine-language instruction in main memory. 
`portal:` [instruction pointers in Crafting Interpreters](<Crafting Interpreters - Chapter 15 Annotation>)

The simplest operations use main memory, the *register file*, and the *arithmetic/logic unit* (ALU). The register file contains a set of word-size registers to hold temporary information, and the ALU computes new data and address values. 

# back to execution
With this simplified hardware model in place, we can get back to what happens when we run `hello.c`:

| event     | effect                                                                                        |
| --------- | --------------------------------------------------------------------------------------------- |
| `./hello` | shell stores `hello` in memory                                                                |
| `enter`   | shell loads `hello` file from disk to main memory                                             |
|           | processor executes `hello`’s instructions                                                     |
|           | instructions copy the bytes in the `"hello, world\n"` string from memory to the register file |
|           | `"hello, world"\n` is copied from register file to the display device                         |
|           | `"hello, world"\n` is displayed on screen.                                                    |

# caching 
It is generally easier to improve the speed of processors than it is to speed up memory accesses. And devices with larger storage are typically slower than those with a smaller storage. 

To deal with this, system designers usually include *caches* that serve as temporary storage for information that a processor will likely need in the near future. 

Caching works by exploiting *locality*, the tendency for programs to access pieces of data that are close to one another in memory. Caches simply store this high-yield information for easy, immediate access by programs. 

>Application programmers who understand cache memories can exploit them to dramatically improve program performance (See [Chapter 6](<CSAPP Chapter 6>)).

Caches are the next level of memory above the register, but the levels continue into an elaborate *memory hierarchy*. As we move deeper into the upper levels of the hierarchy, devices become slower and larger. The idea is that each level is a cache for the level deeper to it. 

# the OS and friends 
Between the program and the hardware is the *Operating System* (OS), which oversees any attempts by the program to manipulate the hardware. 

The OS:
1. protects the hardware from misuse, and 
2. providers apps with simple and uniform methods to manipulate hardware devices. 

The OS achieves both goals via the following fundamental abstractions: 

## processes 
A *process* is an abstraction for a running program. A single CPU can execute multiple processes concurrently via *context switching*, or intelligently switching between processes.

The OS keeps track of any extra information needed for running a process in a *context*, which includes the PC, register file, and main memory contents.

When an OS decides to transfer control from one process to another, it saves and stores the current context, then loads the context of the new process before passing control over to it. 

>when we run `hello` from the shell, the OS saves the shell’s context, passes control to the `hello.c` program, loads its context (which includes the C standard library and standard I/O etc.), and passses control to it. 

Transitions between processes are managed by the *kernel*, a collection of code and data structures that the OS uses to manage all processes. `echo` [[The Kernel|ker]]

Importantly, in modern systems processes can consist of multiple units of execution called *threads*, each of which shares the same context. Explicitly allocating multiple threads to run a program typically speeds programs up substantially (See Chapter 12 for more on concurrency). 
can consist of multiple units of execution called *threads*, each of which shares the same context. Explicitly allocating multiple threads to run a program typically speeds programs up substantially (See Chapter 12 for more on concurrency). 

## virtual memory 
Virtual memory provides each process with a protected view of the main memory called its *virtual address space*. 

For each process, this space consists of the following areas with specific purposes:

1. Program code and data. Discussed in detail in Chapter 7. 

2. Heap. A storage structure for large data structures that expands and contracts dynamically at runtime. 

3. Shared libraries. This space includes the `C` standard library, the math library, and other shared libraries. Discussed in detail with dynamic linking in Chapter 7.

4. Stack. A structure for the compiler to implement function calls that expands and contracts dynamically at runtime. 

5. Kernel virtual memory. A protected enclosure of data that application programs cannot read or write to, and must invoke the kernel to perform any operations on this enclosure.

>**During code execution, the OS stores a process’s virtual memory on the hard disk and uses the main memory as a cache for the disk.** 

This sophisticated interaction between the processor and its hardware will be expanded on in Chapter 9.

## files 
Finally, a *file*, a sequence of bytes, is the basic format that the system uses to represent devices, read inputs and outputs, and perform computations. 

# networks 
All of the preceding ideas apply to a system considered in isolation. In practice, systems are inextricably linked to one another in structure and function via networks. 

A system treats a network as just another I/O device to pass to and receive information from. Email, the worldwide web, FTP, and other applications all rely on the fundamental idea of networks. 

Network apps are discussed in detail in Chapter 11. 

# themes 
The chapter concludes with several recurring central concepts throughout the rest of the book. 

## Amdahl’s Law 
Informally, Amdahl’s Law states that when we speed up one part of a system, the speedup effect on the overall system depends on the component’s importance in the system (i.e. its fractional performance cost relative to the system) and the amount of speedup:

$$
T_{new} = (1 - \alpha)T_{old} + \frac{\alpha T_{old}}{k}
$$
where $\alpha$ is the fraction of $T_{old}$ taken up by the part in question; $k$ is the factor of performance improvement in that part (if $k$ is 3, we made that part 3 times faster).

The speedup $S$, is then expressed as 
$$S = \frac{T_{old}}{T_{new}} = \frac{1}{(1 - \alpha) + \alpha/k} $$

## Concurrency and Parallelism
Concurrency refers generally to a system managing multiple simultaneous activities. Parallelism refers to the use of concurrency to make a system faster. 

Parallelism can be exploited at multiple levels of abstraction: 
1. Threads: at the level of threads, we can allow ,ultiple control flows execute within a single process. 
2. Instructions: at the (lower) level of instructions, modern processors can execute multiple instructions at the same time. This is possible through *pipelining* (discussed in Chapter 4), wherein instructions are cut into stages that can each be processed in parallel, then have their results combined sequentially. 
3. Single-Instruction, Multiple-Data Parallelism: at the lowest level, modern processors can allow a single instruction to perform multiple operations in parallel “under the hood”. This is discussed more deeply in Chapter 5 alongside other optimizations. 

## Abstractions 
[*Abstractions*](<Abstraction>) are interfaces to the operations of a machine that *deliberately hide implementation details* from the user. 

Hiding detail is *extremely* useful in computer science and computer engineering. It allows a single abstraction to be applicable to and unify multiple different kinds of hardware systems. It enables application programmers to adopt a common language for programming their apps, under the faith that any system that runs the app will conform to the same language abstraction used by application programmers. 

>[!aside] A World Without Abstractions
>If we imagine a world without abstractions to computers, every time you wanted to design an app or program for your computer, you would have to open it up, pull out the motherboard, figure out how it works, and design programs to make lights go off on the motherboard in the right order. 
>
>This is a massive waste of time and effort. Abstractions make programming - and using computers in the first place - possible.

The chapter concludes by adding one more abstraction: a *virtual machine*, an abstraction for the entire computer, including its OS, processor, and programs. 

More on this throughout the book. 