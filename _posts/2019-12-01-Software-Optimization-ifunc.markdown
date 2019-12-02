---
layout: post
title:  "ifunc - GNU Indirect Function"
date:   2019-12-01 16:45:00
categories: SoftwareOptimization
tags: GNUC
---
* content
{:toc}

### <strong>ifunc General Idea:</strong>
<strong>ifunc</strong> is a special feature provided by GNUC which can help you to optimize your function call. 
It allows to create multiple implementations of one function. A best implementation can be selected during runtime for particular procesor.

### <strong>ifunc Limintation</strong>
It is limited to to standard C language. It also need all code to be one compiliation. So if you plan to write a software which requires multiple. It also requires all code to be one compiliation. 

### <strong>ifunc Process<strong>
1. get cpu capability
2. populate function pointer tables
3. call function via pointer tables

<strong>Example:</strong>

```c++
void *my_memcpy (void *dst, const void *src, size_t length)
    attribute ((ifunc ("resolve_memcpy")));

    // Return a function pointer
static void (resolve_memcpy (void)) (void)
{
    cpu_features = cpuinfo(); 

    if (cpu_has_neon(cpu_features))
        return &memcpy_neon;
    else if(cpu_has_vfp(cpu_features))
        return &memcpy_vfp;
    return &memcpy_generic_arm;
}
```
