---
title: Rust memory management - CodeSandwich
---
# Introduction
Rust is a young programming language bringing new quality to code. You might have heard about it being fast, secure or easy to implement concurrency with. This introduction is focused on the most important, core feature of Rust: memory management. This system is the main language innovation and most of its unique features are direct consequences of this design.

## Who is this text for and what are the goals
This text is written for people, who are programmers, but don't know Rust or are at the very beginning of learning it. It's easier to understand for readers who know C, C++ or other language with manually managed memory as well as some with garbage collector. It's a high-level introduction intended to present core Rust concepts and encourage further learning. It's not a tutorial, there is no `Hello Rust` in the end.

# Memory management
Modern applications manage their own memory in two main ways: as a stack and as a heap. This may not apply to simpler platforms like embedded systems, but for now let's not focus on them.

## Stack
The stack is expanded and shrinked automatically as program enters and exits certain regions, usually functions, but also loops and branch blocks. All modern, higher-than-assembly-level languages do this automatically. They all behave similarly, programmer demands variable, uses it and then just forgets about it. The compiler knows when memory must be reserved and when cleaned up thanks to the code region boundaries. It's a rigid flow, but it's fast, safe and easy to use.

//PICTURE OF FLOW

## Heap
The heap has more liberal story. Programmer can demand some piece of it from any point in code and then free it in any other point. It's not obviously coupled with program flow and compiler can't tell when and how should it be handled. It's programmer's duty to code it properly.

The memory FIRST must be acquired, THEN used and THEN released exactly ONCE. This seems simple, but mixing it with the rest of application's flow can get tricky and violation of a single step is a catastrophy. Sometimes error will have no consequences, but at other times the application will get terminated, or even worse, its memory will silently become corrupted. The worst part is that this behavior is not deterministic.

//PICTURE OF FLOW

#### Leak
Memory is never released. It becomes a dead weight making application use more resources, than needed. In extreme cases it makes program or even whole system crash if all memory is taken and there is still demand for more.

//PICTURE OF FLOW

#### Use after free
Memory is released, but program still tries to use it. If it was given back to the OS, trying to access it will cause the dreaded segmentation fault and program will be terminated immediately. Another funny part is when released memory is cached by allocator and reused on next acquisition making two random parts of code use same location.

//PICTURE OF FLOW

#### Double free
Memory is released twice. Memory might be given back to the OS and it will terminate program on access. A lot depends on allocator, it can do nothing, release memory in use somewhere else or just crash.

//PICTURE OF FLOW

//TODO

# Traditional solutions
//TODO
## Garbage collector
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
