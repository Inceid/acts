---
tags:
  - project/prog
  - release
date created: 2025-01-03
last updated: 2025-06-29
description: Useful tutorial notes for returning to C programming.

status: stable
---
# Quickstart: the simplest program 

Write this code in a file called `main.c`:
```c
#include <stdio.h>

int main() {
	printf("Hello World!");
	return 0;
}
```

Compile it with this command:
```sh
gcc main.c -o main
```

Run the compiled program:
```sh
./main 
```

You should see `"Hello World!"` appear in your terminal. 

# Basic compilation 
`gcc` or “GNU Compiler Collection” is the standard compiler for `C`. The syntax is 
```sh
gcc main.c <otherlib.c> ... -o main
```

# Pointers and Arrays 
Pointers and arrays are closely connected. 
```c
#include <stdio.h>

typedef unsigned char *byte;

void show(byte start, size_t len) {
	int i; 
	for (i = 0; i < len; i++)
		printf(start[i]);
	printf("\n");
}
```

Note above that `start` is a pointer, but we can dereference the data at its address as if it was an array. The latter applies as well: we can reference array elements with pointer notation. 

`start[i]` indicates that we want to read the byte `i` positions beyond the location pointed to by `start`.

