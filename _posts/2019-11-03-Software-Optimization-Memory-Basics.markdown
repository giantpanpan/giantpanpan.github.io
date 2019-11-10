---
layout: post
title:  "Software Optimization - Memory Basics"
date:   2019-11-03 11:28:00
categories: SoftwareOptimization
tags: memory
---
* content
{:toc}

## Memory Basics
Modern computer store 64 bits (8 bytes) in its every memory location. We can read/write each of byte in each memory location by specifying the address. 

The following structure is very basic and is only fit to the single task system. 
<p><img src="https://giantpanpan.github.io/img/cpu_memory.png" alt="memory"/></p>

When we start to use multi-tasking system, we need to make sure that each of its program/process cannot access to the memory of other program's. Memory abstraction(Virtual Memory) is a way to solve this problem. It can also make each process start from same location when they startup, predict and allocate memory space for that process. 

<p><img src="https://giantpanpan.github.io/img/page_table.png" alt="page_table"/></p>

The virtual memory is a logical memory (software based, not physical) and comprises a virtual address space that exists the address space of the RAM. The virtual address space is divided into pages. Pages are consequtive of blocks of memory with the same size. 

Translation table is used to control the mapping of virtual and actual addresses in physical memory. 


### Three Advantage of Modern Memory Scheme:

1. #### Paging/Swapping
- If the user program need more memory that exceeds page1 and need to access page2, but the translation table has no entry for page2. In this case, CPU will throw an exception, transform the control from the user program to kernel. The kernel will search in the physical memory, figure that page2 is not in use right now, then assign it to the translation table and hook it to the user program.
-  An interesting point is that we don't need to assign actual physical pages to all virtual pages we are going to use. We can assign it when we really need to use it. And if a page is not in use for a while, we can move it data to disk drives and reallocate it to another program. It is so called paging or swapping (exchage between RAM and disk).


2. #### Memory Protection:
- each process has their own memory table/swap
- they cannot see the data stored in the physical memory that is not on their table.


3. #### Deman Loading
- Load couple pages for large program, but only for required features, not need for loading whole software. 

