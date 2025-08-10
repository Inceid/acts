---
tags:
  - project/annotation
  - pcms/cs
  - release
date created: 2025-06-28
last updated: 2025-07-02
keywords:
  - storage
  - ints
  - floats
  - int arithmetic
todo:
  - int arithmetic
aliases:
  - Data and Information
projstatus: v1
---
# Key Points 
- Integers and floats are encoded in “bits” of 0’s and 1’s. 
- Computers do math by manipulating these bits. 
- Some of the rules of regular math *don’t apply* when doing math over bits, and it’s important to know how these rules diverge from what might be expected.

# Definitions 
**Unsigned encodings** represent numbers greater than or equal to 0 in binary bits: 0s and 1s. 
**Two’s-complement encodings** represent integers that are positive or negative. 
**Floating-point encodings** approximate real numbers in the binary system. 

>Ints cover a relatively small range of values but are precise; Floats cover a large range of values but are approximate. 

# Memory 
>Computers are *strict* and *efficient* with how they read and interpret programs.

## Binary, decimal, hex
From a computer’s perspective, there are two letters in the alphabet: `0` and `1`. These are called “bits”.

Every word created from this alphabet must have 8 bits, e.g. $01000010$. Each such word is called a “byte”. Bytes can range from $00000000_2$ to $11111111_{2}$. 
- In decimal notation, this range corresponds to $0_{10}$ to $255_{10}$. 
- Both of these ways of writing numbers suck: writing 8 0-1 digits out is annoying, and converted decimal to binary is annoying. 
- Instead, in practice we use *hexadecimal* or *hex* notation, where values range from $00_{16}$ to $\text{ff}_{16}$. Here, the alphabet is numbers $0$ to $9$ and letters $\text{a}$ to $\text{f}$ - it’s a base 16 number system. Letters take the same value in upper and lower case. `C` denotes hex numbers as starting with `0x`, e.g. `0xff`.

>Each byte has 8 binary digits and each hex digit represents a 4-digit binary number. Therefore, we can represent each byte using 2 hex digits.

See the conversion table below.

![[hexadecimal.png]]

To convert a binary number into hex, first pad with $0$’s to the left until we reach a number of digits that is a multiple of 4. Then convert in groups of 4 as above. 

## Conversion Tricks
### decimal to binary
- we can write any power of 2 $x = 2^n$ by writing $1$ followed by $n$ zeroes.
### decimal to hex
- If $x = 2^n$ and $n = i + 4j$ s.t. $0 \leq i \leq 3$, then $x$ in hex form has:
		- leading digit $2^i$,
		- with $j$ $0$s following. 

>e.g. x = 2,048 = $2^{11}$ has $n = 3 + 4\cdot 2 = 11$, so the hex form is `0x800`.

## General conversion 
Here’s the general case for binary and hexadecimal conversion. 

### decimal to hex 
Divide $x$ by 16 yielding quotient $q$ and remainder $r$. Prepend $r$ to the result. Repeat on $q$. 

### hex to decimal 
multiply each digits by the appropriate power of 16, and sum the result. 

## Size 
Every computer has a word size indicating the max size of a pointer’s address. For a word size of $w$ bits, virtual addresses may range from $0$ to $2^w - 1$ which gives the program access to at most $2^w$ bytes. 

>Using fixed-sized integer types, specifically `int32_t` and `int64_t` in `C`, allows programmers optimal control over data representations. 

## Byte ordering 
There are two conventions for ordering the bytes representing an object. 
- Machines that order bytes with least significant byte first follow the *little endian* convention. The latter, where the most significant byte comes first, is called *big endian*.
- The decision to use one or the other is arbitrary and doesn’t matter for most programmers, but there are specific cases where you should know what conventions for byte ordering that different machines use: 
	1. when transmitting binary data over a network of different machines. See [[CSAPP Chapter 11]].
	2. when interpreting *instruction disassembly*, where bytes encoding instructions for a machine are written in reverse order during program execution. 
	3. when “casting” an object of one type to another. 

## Strings 
A string in `C` is encoded by an array of characters terminated by `\0`, the null character. Each character is encoded by the ASCII character code. 

## Boolean Algebra 
Boolean algebra is a set of mathematical rules for combining binary numbers. By encoding `TRUE` as `1` and `FALSE` as `0`, it captures the basic principles of logical reasoning. 

Claude Shannon first made the connection between Boolean algebra and digital logic in his master’s thesis, where he showed that it was applicable to networks of electromechanical relays. 

# Ints
## Representation
Ints are represented as signed or unsigned in `C`.

An unsigned binary vector $\vec x$ is represented as an integer follows: 

$$
B2U_{w}(\vec x) = \sum \limits_{i=0}^{w-1} 2^i
$$

According to a **Two’s complement** convention, a **signed** binary vector $\vec x$ is represented as an integer as follows:
$$
B2T_{w}(\vec{x}) = -2^{w-1} + \sum \limits_{i=0}^{w-2} 2^i
$$

The name “two’s complement” comes from the fact that the value of a negative number comes from taking a “complement” of a power of 2: you take negative 2 to the power of the most significant bit’s position and add the positive values of the rest of the positions as before.

>[Chapter 3](<CSAPP Chapter 3.md>) introduces instruction generation by **disassemblers**, which disassemble executable programs into machine-readable instructions. These typically include lots of hexadecimal values, typically in two’s complement form. Reading and understanding the meaning of these numbers is an important skill for analyzing/debugging machine code!

## Conversion 
When casting a signed int to an unsigned int, `C` keeps the underlying bit pattern the same but represents its value under the target convention. The same is true vice versa. `C` implements the conversion by converting each case to the binary bit representation, then to the target representation. 

One important relationship is that $UMax_w$ in unsigned form has the same bit representation as $-1$ does in two’s complement form. Equivalently, 

$$
1+ UMax_{w} = 2^w.
$$

This generalizes:

$$
\text{For } x \text{ such that } $TMin_{w} \leq x \leq TMax_{w}: 
$$
$$
T2U_{w}(x) = 
\begin{cases}
    x + 2^w \quad &x < 0 \\
    x \quad &x \geq 0 \\
\end{cases}
$$

Finally, we can derive a closed form for converting two’s complement to unsigned:

$$
T2U_{w}(x) = x + x_{w-1}2^w
$$

Its general behavior is depicted below: 

![[Pasted image 20250630190133.png]]

Negatives are converted to large positives; nonnegatives and smaller positives are unchanged.

>Intuitively, when going from two’s complement to unsigned, the most significant bit changes from negative to positive. so for a word $x$ of length $w$, the sign bit changes its contribution from $-2^{w-1}$ to $+2^{w-1}$. This amounts to a change of $2^w$ in the value of $x$. 

Whereas in unsigned to two’s complement, the following behavior holds instead: 

$$
U2T_{w}(u) = 
\begin{cases}
    u \quad &u \leq TMax_{w} \\
    u - 2^w \quad &u > TMax_{w} \\
\end{cases}
$$

And displays the following behavior:
![[Pasted image 20250630190647.png]]

In closed form: 
$$
U2T_{w}(u) = -u_{w-1}2^w + u
$$

## Resizing 
When expanding a $w$-sized number to size $w' = w + k$, 
- unsigned: we copy $0$s to the left $k$ times. 
- signed (2s complement): we copy $1$s to the left $k$ times.

When truncating a $w$-sized number $x$ to size $w’ = w - k$, we essentially drop the high-order $k$ bits. 
- unsigned: $x’ = x \text{ mod } 2^k$. 
- signed: $x’ = U2T_{k}(x \text{ mod } 2^k)$

In the signed case, this applies $\text{mod}$ then converts the most significant bit $x_{k-1}$ from having weight $+2^{k-1}$ to $-2^{k-1}$. 

## Arithmetic
Understanding nuances of `int` arithmetic can help programmers write more reliable code. 

### Unsigned Addition 

### Two’s Complement Addition

### Two’s Complement Negation 

### Two’s Complement Multiplication 

### Multiplying by Constants 

### Dividing by Powers of 2 


## Summary 
`C` supports both signed and unsigned `int` arithmetic and casting. It does not specify a representation for signed numbers, but most machines use two’s complement. C also does not specify the conversion precisely, but most machines implement it preserving the underlying bit pattern.

>Many languages don’t allow unsigned integers given the ambiguities and potential pitfalls of unsigned arithmetic. But unsigned integers are very useful when we want to think of words as just bit sequences with no numeric interpretation, i.e. when creating flags describing various boolean conditions or masks to modify a bit sequence according to a rule. Addresses are also unsigned.

Note: `C` handles operators containing both signed and unsigned operands by implicitly casting signed to unsigned, then performing the operation assuming the numbers are nonnegative. This can occasionally lead to nonintuitive behavior, e.g. `-1 < 0U` evaluating to `FALSE`.

# Floats 
## Representation 
Floating point representations encode rational numbers of the form $V = x \times 2^y$, and are useful for computations over very large numbers. 

To understand floats, we start with decimal notation: 
$d_m d_{m-1}\cdot\cdot\cdot d_{1}d_{0}.d_{-1}d_{-2}\cdot\cdot\cdot d_{-n}$

which can be wrapped as 
$d = \sum\limits_{i=-n}^{m} 10^i \times d_{i}$.

Digits to the left of the decimal point are weighed by nonnegative powers of 10; digits to the right are weighed by negative powers of 10. 

By analogy, a binary decimal number 
$b_{m}b_{m-1}\cdot\cdot\cdot b_{1}b_{0}.b_{-1}b_{-2}\cdot\cdot\cdot b_{-n+1}b_{-n}$

admits of the form
$b = \sum_{i=-n}^{m} 2^i \times b_{i}$

where the decimal point ‘$.$’ now becomes a **binary point**, to the left of which we weigh digits by nonnegative powers of 2, and the right by negative powers of 2. 

>Shifting the point one position to the left divides the number by 2; a right shift multiplies by 2. 

Recall that computers use finite-length encodings for binary numbers. So they can’t represent certain fractions such as $1/5$ exactly, as it is not of the form $x \times 2^y$. But by adding digits to the right of a decimal point, we can approximate it with increasing accuracy. 

Once numbers get sufficiently large, positional notation becomes cumbersome. So instead, we represent numbers in a form $x \times 2^y$ by giving $x$ and $y$. 

The IEEE floating-point standard represents a number in the form $V = (-1)^s \times M \times 2^E$ as follows: 
- the sign $s \in \{0, 1\}$ determines whether the number is negative or positive. This gets one bit, called the sign bit `s`.
- the significand $M \in [1, 2-\epsilon]$ (or $M \in [0, 1 - \epsilon]$) is a fractional binary number. This gets $n$ bits, which comprise the fractional field `frac` $= f_{n-1}\dots f_{1}f_{0}$
- the exponent $E$ weights the value by a power of 2. This gets $k$ bits, which comprise the field `exp` $= e_{k-1}\dots e_{1} e_{0}$

The above representation can be fitted to a 32-bit representation (1, 8, 23) or 64-bit (1, 11, 52). The value encoded by a given representation divides into three cases: 

### Case 1: Normalized Values
In this case, `exp` is neither all 0s nor all 1s. Then we say `exp` $= e - Bias$, where $e$ is unsigned number $e_{k-1}…e_1e_0$ and $Bias$ is some bias value equal to $2^{k-1} - 1$. 

For single-precision, this yields exponent ranges from -126 to +127; for double, -1022 to +1023. 

`frac` represents $f$ where $0 \leq f < 1$, represented as $0.f_{n-1}…f_1f_0$; we define $M = 1+f$ (we add a leading 1 to the binary $0.f_{k-1}…f_1f_0$).

### Case 2: Denormalized Values 
When `exp` is all 0s, the number is in *denormalized form*. Here, $E = 1 - Bias$, and $M = f$.

Denormalized numbers allow us to represent 0 (in normalized form we must always have $M \geq 1$) and numbers very close to $0.0$. This instantiates a property called *gradual underflow* wherein possible numeric values are spaced evenly near $0.0$. 

### Case 3: Special Values 
When `exp` is all 1s, some special cases arise. 
- When $f$ is all 0s: 
	- `s` $= 0$: the number is $+\infty$. 
	- `s` $= 1$: the number is $-\infty$. 
- When $f$ is nonzero: the number is $NaN$ or “not a number”, usually returned when trying to compute $\infty - \infty$ or $\sqrt{-1}$. 

## Some important properties 
1. $+0.0$ is always all 0s. 
2. The smallest positive denormalized value has 1 as the least significant bit and otherwise all 0s. 
	- Therefore, $M = f = 2^{-n}$ and $E = -2^{k-1} + 2$, giving an overall value $V = 2^{-n-2^{k-1} + 2}.$
3. The largest denormalized value has $E$ as all 0s and $f$ as all 1s. 
	- Therefore, $M = f = 1 - 2^{-n}$ and $E = -2^{k-1} + 2$, giving $V = (1-2^{-n}) \times 2^{-2^{k-1} + 2}$. 
4. The smallest positive normalized value has 1 as the least significant bit of $E$ and otherwise all 0s. 
	- Here, $M = 1$ and $E = -2^{k-1} + 2$. Therefore $V = 2^{-2^{k-1} + 2}$.
5. The value 1.0 has all except the most significant bit of $E$ as 1, other bits as 0. 
	- $M = 1$ and $E = 0$. 
6. The largest normalized value has a sign bit of 0, least significant bit of $E$ as 0, and all other bits as 1. 
	- Here, $f = 1-2^{-n}$, therefore $M = 2 - 2^{-n}$. We have $E = 2^{k-1}-1$, giving V = $(2 - 2^{-n}) \times 2^{2^{k-1}-1} = (1 - 2^{-n-1}) \times 2^{2^{k-1}}$.

## Rounding 
The IEEE floating-point standard defines four *rounding modes* for rounding a number that is halfway between two plausible values (e.g. rounding 1.5 to the nearest whole number): round-to-even, round-to-zero, round-down, and round-up. 
- Round-to-even rounds a number to the nearest number s.t. the least significant digit is even. 
- Round-to-zero rounds positive numbers down and negative numbers up. 
- The others are self-explanatory. 

## Float ops 
For any float $x$ and $y$ and some op $\odot$ over reals, the computation should yield $Round(x \odot y)$. Special cases are handled by conventions that attempt to be reasonable: $1/-0$ is defined as $-\infty$; $1/+0$ is defined as $+\infty$. 

Because rounding is applied to the result of any binary op, addition over floats is commutative but **not associative**. 

>`(3.14+1e10)-1e10 -> 0.0`, but `3.14+(1e10-1e10) -> 3.14`.

All values except infinites and *NaN*s have inverses. 

Multiplication is also not association due to rounding and overflow. For the same reasons, it also does not distribute over addition. Both ops satisfy monotonicity, unlike addition/multiplication over ints. 

## Floats in `C` 
`C` implements floats in two ways: `float` and `double`. These correspond to single- and double-precision floats. Machines use round-to-even rounding. To access special values or change rounding modes, it’s necessary to import special libraries with these features. 

Casting behavior is listed below.

| source          | target | behavior                                                        |
| --------------- | ------ | --------------------------------------------------------------- |
| int             | float  | can be rounded                                                  |
| int or float    | double | exact numeric value is preserved                                |
| double          | float  | value may overflow to $+\infty$ or $-\infty$, or may be rounded |
| float or double | int    | rounds toward 0, may overflow                                   |
