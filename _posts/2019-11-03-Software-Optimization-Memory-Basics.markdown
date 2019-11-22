---
layout: post
title:  "Software Optimization - Memory Basics"
date:   2019-11-03 11:28:00
categories: SoftwareOptimization
tags: memory
---
* content
{:toc}

## <strong>Memory Basics</strong>
Modern computer store 64 bits (8 bytes) in its every memory location. Ideally, We can read/write each of byte in each memory location by specifying the address. 

The following structure is very basic and is only fit to the single task system. 
<p><img src="https://giantpanpan.github.io/img/cpu_memory.png" alt="memory"/></p>

When we start to use multi-tasking system, we need to make sure that each of its program/process cannot access to the memory of other program's. Memory abstraction(Virtual Memory) is a way to solve this problem. It can also make each process start from same location when they startup, predict and allocate memory space for that process. 

<p><img src="https://giantpanpan.github.io/img/page_table.png" alt="page_table"/></p>

The virtual memory is a logical memory (software based, not physical) and comprises a virtual address space that exists the address space of the RAM. The virtual address space is divided into pages. Pages are consequtive of regions of memory with the same size. 

Translation table is used to control the mapping of virtual and actual addresses in physical memory. When a software is trying to access a page, and the page table will show the program where the page is directing to in the RAM. Then, then software can access the data in the corresponding address in RAM. 


### <strong>Advantages of Modern Memory Scheme:</strong>

1. #### <strong>Paging/Swapping</strong>
- If the user program need more memory that exceeds page1 and need to access page2, but the translation table has no entry for page2. In this case, CPU will throw an exception, transform the control from the user program to kernel. The kernel will search in the physical memory, figure that page2 is not in use right now, then assign it to the translation table and hook it to the user program.
-  An interesting point is that we don't need to assign actual physical pages to all virtual pages we are going to use. We can assign it when we really need to use it. And if a page is not in use for a while, we can move it data to disk drives and reallocate it to another program. It is so called paging or swapping (exchage between RAM and disk).


2. #### <strong>Memory Protection:</strong>
- each process has their own memory table/swap
- they cannot see the data stored in the physical memory that is not on their table.
- there are two ways to approach it. one is using only one page table, but each program is allocated into different portition. Another way is each program their own page table.

3. #### <strong>Demand Loading</strong>
- Load couple pages for large program, but only for required features, not need for loading whole software. 
- e.g. if we only need to use a printf function in this program, we just need to call the page that contains printf and ignore the others

4. #### <strong>Per-Process Data</strong>
- Same program call by multiple users, share one executble in memory, but each process gets its own data area.

5. #### <strong>Shared Memory</strong>
- Set up a region of memory, and map it to both of addresses
- multiple process can access same block of memory. Very common on multi-threading programs.  

6. #### <strong>Shared Text Segments</strong>
- Fork() 
    - -> how we create new process in UNIX system <br>
    - -> initially they will have same data

7. #### <strong>Copy-Out-Write(COW)</strong>
- When either of two process try to access to a specific page, we need to make a copy of that point. 

### <strong>Physical Memory</strong>
<p><img src="https://giantpanpan.github.io/img/cpu_memory.png" alt="memory"/></p>

I found I very simple explaination about the relationship between [CPU, cache and memory](https://imgur.com/gallery/aBKD0Fv)

Different cache can talk with each other, to update which is the latest. 

#### <strong>Synchronization & Memory Barriers</strong>
- make sure changes orders are detected
- special instructions: every write/store before this point of time, has to be visiable to every observer, before any writes/store performed after this point of time. It will make sure the lastest value will be propagated accross the entire system before other values get propagated accross the system. It also guarantee the compiler and cpu not to swap these instructions accoss this barrier. 
- x86 force memory ordering (synchronization) accorss the whole system no matter how costs that is all the time. 

## <strong>To Be Continue ...</strong>