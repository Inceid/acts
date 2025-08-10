---
tags:
  - project/annotation
  - release

status: stable
---
# Chapter 4: Scanning.
It's generally good practice to separate the code that generates an error from the code that reports the error.
* That's why, in this case, we put the error reporting code in the lexer, and the error-detecting code in the scanner (which operates when the code is run).
* For error handling, we design a minimalistic token class that includes at least the information that we want to provide the user for debugging errors: line information, the lexeme, literal, and token type of the token at which the error occurs.
The rules that determine how a language groups characters into lexemes are its **lexical grammar.**
We implement the scanner and its grammatical rules primarily using regex matching.

The way we conceptually build up the structure of the scanner is to consider each case of lexeme. How would you recognize lexemes that are one character long? Implement that case. Then implement two characters.

For longer lexemes, a good strategy is to create some lexeme-specific code that keeps consuming characters until it sees the end of the lexeme.
- scanners generally have some number of characters of **lookahead**. This allows them to look ahead in the code that many characters. The smaller this number, the faster the scanner. The amount we need is usually dictated by a language's lexical grammar.

For reserved words and identifiers, and for pattern matching generally, we use the principle of **maximal munch**: when consuming code to match a chunk of code to grammar rules, take as big bites as possible. i.e. match as many characters at a time as you can.