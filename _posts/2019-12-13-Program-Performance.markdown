---
layout: post
title:  "Program Performence"
date:   2019-12-13 18:54:00
categories: SoftwareOptimization
tags: OpenCV
---
* content
{:toc}

## <strong>Program Performance is bound by</strong>
1. <strong>CPU</strong>
2. <strong>Memory</strong>
   - Memory will always prioritize read over write function to CPU, until the cache is running out 
3. <strong>Input and Output</strong>
   - Network Speed
   - Storage 
   - Humans (compare total execution time with total time, if the gap is too large, it means machine is waiting for human input too long time)

## <strong>How to solve performance issues caused by memory Read/Write 
1. <strong>Make CPU job easier</strong>
  - Reduce the size of data (e.g. JSON and html are bulky). 
  - Make data compacted, not let them spread out in memory (memory fragmentation)
  - Localize access (put data we need to access close to each other)
  - Give hints to CPU (memory pre-fetch instructions for both read and write). We can give instructions to CPU, to warn it that we are likely to need a certain piece of data. CPU will go and tell the memory system that it is going to need that. If memory system is not busy, it will go and fetch that data into cache before CPU really need to use that. 

## <strong>Why Move Operator(move from one register to another) can be a hot spot in usage? </strong>
The problem is not move operation, it is the data is not ready yet. Another register is working with this data and it is not free to be move to another register. So the move operation will hang there and wait. So we always need to keep an eye on the time and usage of a software and find out where the hot spot is. If we can solve the hot spot, the overall performance of the software will be better. 