---
layout: post
title:  "Software Optimization - SIMD"
date:   2019-10-30 16:20:00
categories: software_optimization
tags: SIMD
excerpt: Use 
mathjax: true
---

<strong>SIMD</strong> is an acronym for "Single Instruction, Multiple Data". 

When we need to execute a long loop, under certain conditions, we are able to use the SIMD to  vectorize the loop into lanes. Then execute the lanes with same instruction in parallel, to make the execution more efficient. 

For example:

```c++
int an_array[] = {0,1,2,3,4,5,6,7,8};
int total = 0;
for (int i = 0; i < 9; i++)
{
    total += an_array[i];
}

```

Instead of add total in sequential way, we can chunk an_array into 3 lanes, each with three numbers:

```c++
lane1: {0,1,2} = 3
lane2: {3,4,5} = 12
lane3: {6,7,8} = 21
         total = 36
```
Then apply the "+=" intruction for three lanes in parallel, for each we can get the result 3,12,21. After that, SIMD will summerize three temp result.

Modern computer provides vector(SIMD) registers with range from (128 bits to 2048 bits) to store multiple lanes of similar data. 

A 128 - bit vector register can be used as

- 2 64-bit lanes
- 4 32-bit lanes
- 8 16-bit lanes
- 16 8-bit lanes

We need notice that for different architecture, e.g. AArch64 and x86-64, the notation for vector registers. 

Aarch64 vector usage - vn.s:
- n -> register number
- s -> shape of the lanes

v0.16b means vector registor use 16 lanes of 8 bits(1 byte) each.
v8.4s means registor 8 use 4 lanes of 32 bits (single-word) each

Execution:
```bash
$add v0.8h, v1.8h, v2.8h
```
add the value in first lane of register 1 and the value in first lane of register 2, and place the result of to first lane of register 0. 8 times of this operations will be implemented. 

Aarch64 scalar usage - sn:

q3: vector registor3 use a single 128-bit value

<strong>Three ways to use SIMD</strong>
1. <strong>gcc</strong> compiler option
gcc with <strong>-O3</strong> (max optimization level) will turn on <strong>-ftree-vectorize</strong> flag by default. <strong>-fopt-info-vec-all</strong> can show you the compiler decisions about vectorization. This flag will automatically apply vectorization if certain conditions meet:
- number of loop iterations is known before loop start
- memory layout meet SIMD allignment 
- vectorization must not affect the result of regular loop execution
- the benefit of using vectorization is begger than its cost. If the loop is very small, there is no need to set up vectorization.
2. Inline Assembler
It is architecure-dependent. We need to use C code like following to check the arch first then do the specific vectorization:


```cpp
#if __x86_64__
// x86 vectorization format
#endif

#if __aarch64__
// aarch64 vectorization format
#endif
```

3. C Intrinsics:
C language function-like extensions. Link to outside SIMD instructions.