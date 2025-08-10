---
tags:
  - postmortem
  - project
  - release
date created: 2024-11-30
last updated: 2024-12-20

status: stable
---
# Primordia
For the next time we do a guided project that will serve as a basis for future projects, it will be good to explicitly version that project. For clox, it will be good to create major versions of clox with core feature implementation changes over each major version. This benefits archival, as it allows us to see the progression of our project over time, which forms a precedent for how we implement large-scale coding projects in the future.

# Usage
Our implementation of `jlox` implements all of the functionality described in *Crafting Interpreters*: Chapters 1-13. 

This repo contains an executable `lox.jar` which can execute and print/produce any output of a script written in Lox's syntax. `lox.jar` is located in lox/out/artifacts/lox_jar.

To recompile an updated `lox.jar` after a code update, go to InteliiJ's Menu -> Build -> Build Artifacts -> Edit -> Name the output lox:jar (if doesn't exist) -> Specify the output dir -> Ensure file type is JAR -> Click OK -> go back to Build -> Build Artifacts -> Build.

After adding the following alias to your terminal, you can run `jlox` in interactive mode by typing `jlox` or run a file by typing `jlox <filename>`.

```bash
alias jlox='java -jar /mnt/c/Users/suraj/IdeaProjects/lox/out/artifacts/lox_jar/lox.jar'
```
