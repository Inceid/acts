---
tags:
  - project/annotation
  - release
date created: 2025-02-03

status: stable
---
# Garbage Collection 
We implement garbage collection (GC) using a conservative approach: any variable or object that *could* be referenced by the user’s program should be preserved. That implies that any variable or object that *can’t* be referenced by the user is fair game for deletion by the garbage collector.

We say an object is “reachable” if there’s some chain of references that lets the user access it throughout the program. So reachable objects are untouched, and unreachable objects are all freed.

This translates to a recursive algorithm:
1. Starting with “roots” or directly accessible objects, traverse through object references to find all reachable objects. 
2. Free all objects not in that set.

In practice, this is implemented as a **mark-sweep** algorithm. The algorithm has basically two steps, similar to the above. First, use a graph-like search starting from root values through any references encountered through them and **mark** any values you encounter. Then, **sweep** (that is, free) any objects not marked.

In our implementation, the roots that we have to track include local variables/temporaries on the stack, objects on the heap, [open upvalues](<Crafting Interpreters - Chapter 25 Annotation>), and the compiler’s currently (and enclosing) compiled function(s).

So much for roots. To track references, we use a “tricolor algorithm” as follows.

# Tricolor Abstraction
We use three colors to denote the state of an object according to the GC.

* If the GC hasn’t seen an object yet, it’s white.
* If the GC has seen an object but hasn’t processed its references (or checked to see if it has references) yet, it’s gray. 
* If the GC has fully processed an object, it’s black. 

The algorithm to mark references proceeds as such.
1. Mark all objects white. 
2. Traverse all roots and mark them gray. 
3. While there are gray objects, take the next gray object and mark its references gray. Mark it black.
4. Free any white objects. 

In our implementation, all objects start unmarked (white) and are added to a “graystack” or worklist of objects to be processed. Once they are processed, they are marked (“blackened"). Then, once we have finished blackening all objects, we sweep every object still white. 

# Optimizing Throughput and Latency
Lastly, we make a couple of provisions to improve throughput and latency. 

Throughout is the total amount of time in program execution actually spent running the user’s code (as opposed to garbage collecting). Latency is the longest contiguous period during which a GC is running. 

GC implementations necessarily make tradeoffs between maxxing throughput and minning latency, but it’s clear we generally want a balance of both. We achieve this by setting a threshold `nextGC` to render the heap self-adjusting: when we allocate `nextGC` number of bytes onto the heap, we trigger a GC and update `nextGC` to be double (or any coefficient multiple that makes sense) the size after collecting. 

This allows us to dynamically call the GC when it makes the most sense - frequently enough to not let too much unreachable memory pile up, but not too frequently such that the GC is revisiting live objects incessantly.

# Design Miscellany 
Some important design considerations:
- We call our GC *every time we allocate memory*. This implies that, even during compilation (wherein the compiler needs to allocate memory to store temporaries and upvalues), the GC may be called several times. In this case, we need to ensure the compiler has marked its own roots and references as safe before the GC eats them up. 
- Since we intern our strings into a hashset by means of **weak references** (these are pointers that do not protect an object from garbage collection), we need to treat them specially during GC. 
	- `q:` Why GC them at all? `a:` Our intern table has references to strings, but these are not necessarily references the user’s program can reach, so we want string interns to function as unreachable by default (unless a variable assigns to them, etc.) 
	- The problem: once an intern is collected, its pointer will still exist and point to freed memory (i.e. `NULL`). This is bad if undealt with. We deal with this by traversing the table and fully deleting any unmarked interns.
	* Importantly, we do this *in between* tracing gray references and sweeping white objects. That lets us catch unreachable interns 