---
layout: post
title:  "OpenCV Optimization Practice - Stage II"
date:   2019-11-30 13:40:00
categories: SoftwareOptimization
tags: OpenCV
---
* content
{:toc}

# <strong>OpenCV Optimization - Stage II </strong>

<p><img src="https://giantpanpan.github.io/img/opencv.png" alt="OpenCV"/></p>

In this blog, I will continue to work with openCV fast-math functions optimization. You can look at the source code of  [OpenCV](https://github.com/opencv/opencv) here. 

### <strong>Optimization Opportunities</strong>

<strong>Fisrt of all, let's have a breif review about what I did in openCV optimization stage I.</strong>

In stage one, I found that I maybe able to optimize the cvFloor and cvCeil funtions for AArch64. According to the ```perf report``` result:

```
  35.47%  main     main                        [.] cvCeil                      ◆
  13.00%  main     main                        [.] cvRound                     ▒
```

The usage percentage of cvCeil is almost 3 times higher than cvRound. I think the logic of ceil function should be similar or even simpler than round ...?
But why the cvCeil performance is much worse than cvCeil?

I guess the reason is OpenCV developers used an optimzed methods for cvRound function:
```c++
CV_INLINE int
cvRound( double value )
{
    ...
#elif defined CV_ICC || defined __GNUC__
    return (int)(lrint(value));
    ...
}
```

But in cvCeil side, it will use the regular method:
```c++
CV_INLINE int cvCeil( float value )
{
    ...
#else
    int i = (int)value;
    return i + (i < value);
#endif
}
```

### <strong>Another Interesting Discovery</strong>
If you have read my OpenCV optimization stage I blog, you might recall that the initially, I plan to optimize cvRound. But cvRound seems already optimized.
I discussed my concern with my instructor and showed him this source of fast_math functions. He mentioned me that the cvRound optimization methods for x86_64 looked odd.     

It is coded like this:
```c++
cvRound( double value )
{
    ...
#elif ((defined _MSC_VER && defined _M_X64) || (defined __GNUC__ && defined __x86_64__ \
    && defined __SSE2__ && !defined __APPLE__) || CV_SSE2) \
    && !defined(__CUDACC__)
    __m128d t = _mm_set_sd( value );
    return _mm_cvtsd_si32(t);
    ...
```
It seems like they are using some magic here :0
I did some research about this block of code. OpenCV developers here use Intel Intrinstics functions for optimization.


``` __m128d ``` <br>
Description:
- representing a 128-bit SIMD register which internally is consisted of two packed f64 instances. Usage of this type typically corresponds to the sse and up target features for x86/x86_64.


```__m128d _mm_set_sd (double a)```<br>
Synopsis:
- __m128d _mm_set_sd (double a)
-  #include <emmintrin.h>
- CPUID Flags: SSE2

Description:
-   Copy double-precision (64-bit) floating-point element a to the lower element of dst, and zero the upper element.

```int _mm_cvtsd_si32 (__m128d a)```<br>
Synopsis:
-    int _mm_cvtsd_si32 (__m128d a)
-   #include <emmintrin.h>
-  Instruction: cvtsd2si r32, xmm
- CPUID Flags: SSE2

Description:
- Convert the lower double-precision (64-bit) floating-point element in a to a 32-bit integer, and store the result in dst.

So basically, this optimization method is going to store a 64-bit floating point into a 128-bit SIMD register. The floating point will be stored in the lower element of this register, and zero in the upper element. Finally, it will convert the lower element into a 32-bit int and store the result to destination.

??? Why we need 128 bit SIMD resgister to store a float? It seems more complicated and resoure-wasting than the basic logic of rounding. Maybe OpenCV developers has reasons to do so and I did not see it yet. Anyway, I will leave a question mark here and come back later. 

### <strong>Optimization Approaches</strong>
<strong>After previous benchmarking and profiling, I decide to use two approaches</strong>

1. Baseline approach - use optimized functions, similar with ```lrint(value)``` for cvCeil and cvRound
2. Advanced approach - use Inline Assembly Language for cvCeil and cvRound and see whether it will be better than first approach
 
#### <strong>1. Use Optimized Functions</strong>

I choose to use ```ceil(value)``` and ```floor(value)``` functions. These are all c++ functions. They will convert the input value to the corresponding ceiled/floored result with same data type. Then I will convert the result to integer and return it.

```c++
CV_INLINE int cvCeil( double value )
{
#if (defined CV__FASTMATH_ENABLE_GCC_MATH_BUILTINS || defined CV__FASTMATH_ENABLE_CLANG_MATH_BUILTINS) \
    && ( \
        defined(__PPC64__) \
    )
    return __builtin_ceil(value);
#elif __aarch64__
    return (int)(ceil(value));
#else
    int i = (int)value;
    return i + (i < value);
#endif
}
```

```c++
CV_INLINE int cvFloor( double value )
{
#if (defined CV__FASTMATH_ENABLE_GCC_MATH_BUILTINS || defined CV__FASTMATH_ENABLE_CLANG_MATH_BUILTINS) \
    && ( \
        defined(__PPC64__) \
    )
    return __builtin_floorf(value);
#elif __aarch64__
    return (int)(floor(value));
#else
    int i = (int)value;
    return i - (i > value);
#endif
}
```

##### <strong>The testing code</strong>

```c++
int main(int argc,char** argv){
if( argc != 2 )
    {
#if __aarch64__
    printf("__aarch64__\n");
#endif
    //double a = 23.776817263;
    for (int i=0;i<10000000;i++){
        cvRound(2.5);
        cvCeil(2.5);
        cvFloor(2.5);
    }
    return 0;
    }
}
```

##### <strong>Result</strong>
The ```perf report ``` shows us that the performance of cvFloor and cvCeil is very close to cvRound right now. Comparing with the result before this approach, I think this imporvment is decent. 

```
  16.96%  main     main                          [.] cvFloor                   ▒
  14.99%  main     main                          [.] cvCeil                    ▒
  13.61%  main     main                          [.] main                      ▒
  10.69%  main     libm-2.27.so                  [.] __lrint                   ▒
   9.89%  main     main                          [.] cvRound                   ◆
   8.84%  main     ld-2.27.so                    [.] do_lookup_x               ▒
   4.58%  main     main                          [.] lrint@plt                 ▒
   4.05%  main     main                          [.] floor@plt                 ▒
   4.02%  main     libm-2.27.so                  [.] __ceil                    ▒
   3.73%  main     main                          [.] ceil@plt                  ▒
   3.67%  main     libm-2.27.so                  [.] __floor                   ▒
   0.98%  main     ld-2.27.so                    [.] strcmp                    ▒
   0.87%  main     ld-2.27.so                    [.] _dl_lookup_symbol_x 
```

Let us look closer to ```cvFloor```, ```cvCeil``` and ```cvRound```:
```
Percent│
       │
       │    Disassembly of section .text:
       │
       │    00000000004067c0 <cvFloor(double)>:
       │    _ZL7cvFloord():
 17.17 │      stp    x29, x30, [sp, #-32]!
  9.13 │      mov    x29, sp
 11.57 │      str    d0, [sp, #24]
  9.52 │      ldr    d0, [sp, #24]
       │    → bl     floor@plt
 30.59 │      fcvtzs w0, d0
  5.79 │      ldp    x29, x30, [sp], #32
 16.23 │    ← ret
9, sp
 11.57 │      str    d0, [sp, #24]
  9.52 │      ldr    d0, [sp, #24]
       │    → bl     floor@plt
 30.59 │      fcvtzs w0, d0
  5.79 │      ldp    x29, x30, [sp], #32
 16.23 │    ← ret

```

```
```

```
Percent│
       │
       │    Disassembly of section .text:
       │
       │    00000000004067e0 <cvCeil(double)>:
       │    _ZL6cvCeild():
 10.77 │      stp    x29, x30, [sp, #-32]!
  8.23 │      mov    x29, sp
 13.50 │      str    d0, [sp, #24]
  8.66 │      ldr    d0, [sp, #24]
       │    → bl     ceil@plt
 32.72 │      fcvtzs w0, d0
  8.44 │      ldp    x29, x30, [sp], #32
 17.69 │    ← ret
```

```
Percent│
       │    Disassembly of section .text:
       │
       │    00000000004067a4 <cvRound(double)>:
       │    _ZL7cvRoundd():
 11.82 │      stp    x29, x30, [sp, #-32]!
 13.76 │      mov    x29, sp
 23.28 │      str    d0, [sp, #24]
 10.56 │      ldr    d0, [sp, #24]
       │    → bl     lrint@plt
 13.44 │      ldp    x29, x30, [sp], #32
 27.14 │    ← ret

```
The difference between ceil, floor and lrinf is that the lrint will convert the float to integer inside of __lrint (the main body of lrint), but the this process will be excuted inside of ceil and floor functions. So, in total, the performance of these three functions should be very similar.

