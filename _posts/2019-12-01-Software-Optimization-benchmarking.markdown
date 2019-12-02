---
layout: post
title:  "Important Issues for Benchmarking"
date:   2019-12-01 19:42:00
categories: SoftwareOptimization
tags: GNUC
---
* content
{:toc}

### <strong>What is benchmarking</strong>
Benchmarking is an important tool we use to access the relative performance of a software or specific component of a software, normally by running a number of standard tests and trials against it. 





### <strong>Issues we need to notice for benchmarking</strong>
1. Repeatability
    - ability to produce same results reliably (same data, multiple runs)
2. Test conditions
    - make sure we use same testing data, platform and environment
3. Data size and & data composition
    - use enough data to get a decent run, especially using sampling sampling profilers. 
    - use data representative of real-world, ensure code path coverage.
4. Caching - first run code will load up the cache, subsequent run will faster
    - run more than one time
    - or flush your cache everytime
5. Avoid including scaffolding/unrelated functions in benchmarking
    - benchmark the overhead seperately or substract it
    - benchmark only important pieces

