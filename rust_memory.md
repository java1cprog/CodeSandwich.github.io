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

### Leak
Memory is never released. It becomes a dead weight making application use more resources, than needed. In extreme cases it makes program or even whole system crash if all memory is taken and there is still demand for more.

//PICTURE OF FLOW

### Use after free
Memory is released, but program still tries to use it. If it was given back to the OS, trying to access it will cause the dreaded segmentation fault and program will be terminated immediately. Another funny part is when released memory is cached by allocator and reused on next acquisition making two random parts of code use same location.

//PICTURE OF FLOW

### Double free
Memory is released twice. Memory might be given back to the OS and it will terminate program on access. A lot depends on allocator, it can do nothing, release memory in use somewhere else or just crash.

//PICTURE OF FLOW

# Traditional solutions
The heap management problem is very old and programmers invented many tools to mitigate it. There are two main ways, both proven useful, but each of them severely flawed.

## Garbage collector
This is the easy way. The program gets special mechanism detecting moment, from which given memory chunk will never be used, so it can be safely released. This prevents leaking, using after freeing and double freeing. The easiest method of proving that memory will never be used again is proving that it's not reachable. Memory is said to be reachable when program can obtain its address without guessing. It means that it's stored somewhere on stack, in a static variable or in place on heap, which itself is reachable.

//PICTURE OF REACHABILITY

There are numerous smart strategies of checking reachability, but they all generate a significant overhead. For example reference counters increase memory usage and add overhead for each heap access. On the other hand tracing garbage collectors allow free access, but introduce heavy memory reachability analysis, which can either be constantly running in background or it can  completely stop program execution for clean-up. No matter what, garbage collectors add extra work for applications and increase their memory usage.

## Strict disciplines
So garbage collector is a good, but heavy solution. But what can be done when its cost is unbearable or there is just no possibility of using it? Programmers have invented a special discipline, which closely followed makes proper memory management much easier. It's based on the rules of ownership and lifetimes.
 
### Ownership discipline
Ownership is an idea, that there can be many pointers to allocated memory, but only one of them is considered owning. When it's destroyed, it should be used to release the allocation. Non-owning pointers can be created and destroyed in any number, but they should never be used to deallocate. This makes memory management much clearer, because there is only one important pointer to follow and release. It also solves two of three heap problems: leaking and double free. The ownership may be a soft agreement in API and program flow, but some languages and libraries provide tools making execution of this policy more explicit and less error-prone. For example raw C has no such tools, but modern C++ provides built-in smart pointers, which explicitly represent owned pointers and implement proper behavior like deallocation on destruction.

### Lifetimes discipline
Lifetime is a span of time during program execution, when particular piece of data is valid to be used. It's a critical property when dealing with heap allocated memory pointers, which are not owning. They are safe to use as long as the owning pointer doesn't release. After that it's an error to use them, so their lifetimes are over. It's also worth noting, that any structure containing pointer with given lifetime should be considered having lifetime no longer than pointer's. This is not an easy discipline to execute, but it prevents the third heap memory problem: use after free. This complements ownership's guarantees making program fully memory safe without garbage collector overhead.

//PICTURE OF LIFETIMES

# Rust
Rust is sometimes described as a hybrid solution. Actually all it does is enforcing the ownership and lifetimes discipline in code, but the result is that working with Rust is so safe and carefree, that it resembles garbage collected language. Compiler statically verifies, that program is memory safe and if it fails to do so, it generates an error pointing out potential risks. When it passes, the code is guaranteed to never cause memory corruption. Because it all happens BEFORE building the output binary, the process has no influence on it, it's as lightweight as if it was written in pure C or C++.

## Ownership
Rust has very strict notion of ownership. Every allocated piece of memory is owned by single instance of some structure. The structure could be anything, but usually they end up being some kind of collections or boxes (Rust's smart pointers) from the standard library. These wrappers are responsible for deallocating memory, when they are destroyed. There is no easy way to explicitly allocate memory and get a raw pointer without any responsible wrapper.

//PICTURE OF MEMORY OWNERSHIP

### Recursive destruction
Ownership is recursive, so if one structure stores another by value, it gains ownership of the latter and all of its sub-structures. This also means, that when container is destroyed, it must recursively destroy all its content. Rust does it out of the box. All structures have defined destructor, which iterates through all fields and destroys them first. Structure author can add own steps during destruction, for example close DB connection when writing client, but fields still will be destroyed one by one after that. The default behavior is just right in vast majority of cases, so structures rarely have destructors defined and no mater if they do or don't, they are all safe from leaking memory.

//PICTURE OF RECURSIVE DROP

### Tying heap with stack
Rust's ownership model brings a powerful feature: complex heap management is reduced to simple stack management. Programmer doesn't have to worry about allocating and releasing, it's all reduced to working with local variables. Even if structures are deeply nested with many steps of references to heap memory, there always is a single root structure on stack, which will be automatically destroyed when program stops caring about it.

//PICTURE OF COMPLEX STRUCTURES

## Lifetimes
Unfortunately it's not convenient to write programs, where accessing data requires owning it. Rust has normal, not smart references, which make this possible. When such reference is created, it's said, that the value is borrowed. Borrowing creates a two-way relationship: the reference must have lifetime no longer than the value, but the value must not be moved during reference's lifetime. If any of these rules is violated, reference starts pointing at invalid memory. Rust tracks and enforces lifetimes correctness statically and rejecting unsafe configurations.

//PICTURE OF LIFETIME CORRECTNESS

### Recursive borrowing
Structures can never outlive any of their fields. If one of them happens to be a reference, the whole structure instance must be proven to be destroyed before the referenced value. If there is a reference to structure with lifetime constraint, the reference itself must live no longer than the structure. This relation can be nested and tightened any number of times as long as the compiler can prove it safe. When compiler can't guess right relations, they can be explicitly defined with simple grammar.

//PICTURE OF NESTED STRUCTS

## Bending the rules
It would be naive to think, that every system can be expressed in code with that restrictive, statically proven safety. In vast majority of cases it can be done with no effort, but sometimes the rules must be bent and Rust provides tools for doing that. 

### Wrappers
Standard library provides some wrappers pushing ownership and borrowing check to runtime. This calms down validity checker and gives flexibility with little runtime overhead. For example Rc is a box (allocation with smart pointer) with no owner. It's a garbage collected memory with reference counter, which is destroyed with last reference. Rust offers more wrappers, but they are slightly outside of scope of this introduction, they push to runtime other rules, which were not covered here.

### Unsafety
Rust's safety guarantees become impossible to apply when going sufficiently low level in libraries and tooling. For example box and collections are touching memory allocations and pointers without safety guarantees, because THEY ARE safety guarantees. They can be written in Rust, because their code is explicitly marked as unsafe. It lets completely ignore safety checks, but it's very dangerous. All external C library wrappers at some level must use unsafe code as well. They define safety rules making them seamlessly integrated with the rest of code. Unsafe code is Rust's source of power, but also responsibility, it should be avoided at all cost.

# Reality
This system looks good on paper, it's designed by smart people using academic research of other smart people, but is it really useful? Yes, it is. Most of the time it only forces sensible and safe design with explicit relations between elements. After all Rust was designed in parallel with Servo, a future engine of Firefox web browser. From it's very beginning it wasn't only theoretically fine, it was proven usable in real, complex software development. After over a year of commercial programming in Rust I can confirm myself, that Rust's rules are not burden, but great help in architecture and correctness guarantee. I honestly believe, that this language is the future, I can't recommend it more.
