---
layout: post
title:  "Software Optimization - SIMD Practice"
date:   2019-10-30 23:32:00
categories: SoftwareOptimization
tags: SIMD gcc Optimization Architecture
excerpt: Use 3 SIMD vectorization methods to optimize the software
mathjax: true
---

As we discussed in previous blog <a href="../Software-Optimization-With-SIMD">Software-Optimization-With-SIMD.</a>
We are going to use three methods to see the performance of SIMD vectorization.

1. gcc compiler option

```c++
// Fill the array with random data
for (x = 0; x < SAMPLES; x++) {
    data[x] = (rand()%65536)-32768;
}

Result: vol1.c:25:2: note: not vectorized: loop contains function calls or data references that cannot be analyzed
```

```c++
// Scale the volume of all of the samples
for (x = 0; x < SAMPLES; x++) {
    data[x] = scale_sample(data[x], 0.75);
}

Result: vol1.c:32:2: note: LOOP VECTORIZED
```

```c++
// Sum up the data
for (x = 0; x < SAMPLES; x++) {
    ttl = (ttl+data[x])%1000;
}

Result :vol1.c:38:2: note: not vectorized: unsupported use in stmt.
```

After use -O3 and -ftree-vectorize options, we are going to see the auto vectorization results from compiler. Among these three loops, only the second loop can be successfully vectorized. 

