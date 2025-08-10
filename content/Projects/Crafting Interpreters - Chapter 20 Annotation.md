---
tags:
  - project/annotation
  - release

status: stable
---
# Hash Tables
We implement Hash Tables. 

Hash Tables are a prerequisite for implementing variables, as they allow us to look up a variable’s value using its name. 

Hash tables are extremely powerful because they offer **constant time lookup** regardless of the number of keys in the table. 

# The Basic Structure 
A hash table consists of a dynamic array, a hash function, and operations to insert, access, and delete from the table. 
- The arrah 
# Hashing
For performance’s sake, we eagerly cash the hash value of a string. This makes sense as (1) strings are immutable, so that value’s never changing, and (2) we never have to recalculate it for a given string at lookup time.

For brevity’s sake (and only decent-performance requirements for Lox’s purposes), we use the FNV-1a hash algorithm.

# Design Miscellany 
## Tombstones are clever 
Tombstones end up being more time-efficient than manually reinserting all entries affected by a deletion to preserve probe sequence. The reason seems to be that they make deletion *lazy*: it does as little work as is needed to preserve the table’s invariants. That penalizes lookups slightly, which end up checking and skipping over tombstones wastefully. 

But this allows the tombstone to be used by a later insertion, which is a cheap way of avoiding rearranging the affected entries. Under the assumption that this node may be used again soon, you give it a dummy value to keep the probe sequence rather than demolishing and rebuilding the probe sequence anew. Smart!