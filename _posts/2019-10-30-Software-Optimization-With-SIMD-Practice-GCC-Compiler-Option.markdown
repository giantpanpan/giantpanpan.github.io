---
layout: post
title:  "Software Optimization - SIMD Practice (gcc compiler option)"
date:   2019-10-30 23:32:00
categories: SoftwareOptimization
tags: SIMD gcc Optimization Architecture
---

As we discussed in previous blog <a href="../Software-Optimization-With-SIMD">Software-Optimization-With-SIMD.</a>
We are going to use three methods to see the performance of SIMD vectorization.

## gcc compiler option:

```c++
//Loop 1
// Fill the array with random data
for (x = 0; x < SAMPLES; x++) {
    data[x] = (rand()%65536)-32768;
}

Result: vol1.c:25:2: note: not vectorized: loop contains function calls or data references that cannot be analyzed
```

```c++
//Loop 2
// Scale the volume of all of the samples
for (x = 0; x < SAMPLES; x++) {
    data[x] = scale_sample(data[x], 0.75);
}

Result: vol1.c:32:2: note: LOOP VECTORIZED
```

```c++
//Loop 3
// Sum up the data
for (x = 0; x < SAMPLES; x++) {
    ttl = (ttl+data[x])%1000;
}

Result :vol1.c:38:2: note: not vectorized: unsupported use in stmt.
```

After use -O3 and -ftree-vectorize options, we are going to see the auto vectorization results from compiler. Among these three loops, only the second loop can be successfully vectorized. 

The reason Loop 2 is able to be vectorized is that the single intruction ```scale_sample(data[x], 0.75);``` can be applied to various lanes and the result of lanes will not be influenced by order of executions. 

Loop 3 is failed to be vectorized because the instruction ```ttl = (ttl+data[x])%1000;``` is order sensitive and the result of each current ttl is depend on the previous ttl. We cannot simply do same instruction on each lane and sum them up at the end. 

Let's make an minimal example:

```c++
ttl = 0;
data[4] = {1,2,3,4};
```

In sequencial way, each loop should be:
```c++
ttl = (0+1)%1000 = 1
ttl = (1+2)%1000 = 3
ttl = (3+3)%1000 = 6
ttl = (6+4)%1000 = 10
```

If we seperate it into 2 lanes and apply this operation:
```c++
//Lane 1
ttl = (0+1)%1000 = 1
ttl = (1+2)%1000 = 2

//Lane 2
ttl = (0+3)%1000 = 3
ttl = (3+4)%1000 = 7
```
The result of serial version and vectorization can be totally different. So the compiler decise not to use auto-vectorization.

In order to optimize the loop3, we can make this loop vectorized by move the remainder outside


```c++
//Loop 3
// Sum up the data
for (x = 0; x < SAMPLES; x++) {
    ttl = ttl+data[x];
}

Result :vol1.c:38:2: note: LOOP VECTORIZED.
```

In this case, the result each lane can be independent from others after we copy ```ttl = 0``` to each lane. we need to notice that the ttl result will be affected.

To see examples about C Inline methods, please go to next blog: <a href="../../31/Software-Optimization-With-SIMD-Practice-C-Inline">Software-Optimization-With-SIMD-Practice-C-Inline</a>