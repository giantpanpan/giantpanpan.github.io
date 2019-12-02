---
layout: post
title:  "Software Portability and Atomics"
date:   2019-12-01 20:32:00
categories: SoftwareOptimization
tags: GNUC
---
* content
{:toc}

### <strong>Software Portability</strong>
<strong>When we talk about software portability, we usually come up with 2 categories</strong>
1. portable software - cross-platform software, which can be transport from platfrom to platform 
2. port software - software adopt to one platform





<strong>There are four dimensions determining software portability</strong>
1. Hardware
2. Language
3. Libraries
4. Operatio  System / Interfaces

For example LP64 and ILP32 are two different C Language Data Type Models. 
If we define  ```size_t a; ``` in ILP32, its size will be 32 bits. But if we define it in LP64, it will be 64 bits. 

As a result, if we want to develop a portable software accross platforms, we have to think about how to overcome these variations.

### <strong>Atomics</strong>
Atomic is a physics idea, in software development point of view, it means to make operations indivisiable/uninterruptible. The reason is that sometimes, multiple threads will interrupt one operation when it is executing in multi-core situations.

<strong>For example:</strong>
```c++
if (x == 0)
{
    x += 1;
}
```
It is possible that other thread interruption will happen between the addition and assignment process. 

To avoid this problem, we can use intrinstics:
e.g. 
```c++
__atomic_load_n
__atomic_store_n
```

An atomic operation can both constrain code motion and be mapped to hardware instructions for synchronization between threads (e.g., a fence). 