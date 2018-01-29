---
title: Rust memory management - CodeSandwich
---
# Introduction
Rust is a young programming language bringing new quality to code. You might have heard about it being fast, secure or easy to implement concurrency with. This introduction is focused on the most important, core feature of Rust: memory management. This system is the main language innovation and most of its unique features are direct consequences of this design.
## Who is this text for and what are the goals
This text is written for people, who are programmers, but don't know Rust. It's advised for readers to know C, C++ or other language with manually managed memory as well as some with garbage collector. It's a high-level introduction intended to present core Rust concepts and encourage further learning, definitely not a tutorial.
# Program memory management
Modern applications manage their memory in two main ways: as a stack and as a heap. This may not apply to simpler platforms like embedded systems, but for now let's not focus on them.

## The stack
The stack is expanded and shrinked automatically as program enters and exits certain regions, usually functions, but also loops and branch blocks. All modern, higher-than-assembly-level languages do this automatically. They all behave similarly, programmer demands variable, uses it and then just forgets about it. The compiler knows when memory must be reserved and when cleaned up thanks to the code region boundaries. It's a rigid flow, but it's fast, safe and easy to use.

## The heap
The heap has more liberal story. Programmer can demand some piece of it from any point in code and then free it in any other point. It's not obviously coupled with program flow and compiler can't tell when it should be cleaned up.

## The problem


//TODO
# Traditional solutions
//TODO
## Garbage collector
//TODO
## No memory management
//TODO
## No memory management + ownership + lifetimes
//TODO
# And here comes Rust
//TODO
## Ownership as part of grammar
//TODO
### Borrowing
//TODO
## Lifetimes as part of grammar
//TODO
## Are stack and static lifetimes enough?
//TODO
### Usually yes
//TODO
### Workaround wrappers
//TODO
# Results
## No runtime
## No world stops
## Usable everywhere
//TODO
