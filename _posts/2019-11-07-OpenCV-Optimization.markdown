---
layout: post
title:  "OpenCV Optimization Practice - Stage I"
date:   2019-11-07 10:50:00
categories: SoftwareOptimization
tags: OpenCV
---
* content
{:toc}

# <strong>OpenCV Optimization - Stage I </strong>

<p><img src="https://giantpanpan.github.io/img/opencv.png" alt="OpenCV"/></p>

After some experience of previous software optimization practices, I decide to play with to a very popular computer vision library - [OpenCV](https://github.com/opencv/opencv) to see whether the optimization skills I learned before can make a little bit improvement to a real-world project.



### <strong>Before Start</strong>

<strong>Fisrt of all, we need a brief [introduction about OpenCV](https://opencv.org/about/).</strong>

In general speaking, OpenCV (Open Source Computer Vision Library) is an open source computer vision and machine learning software library. It provides provides more than 2500 optimized algorithms. Users can easily uses these algorithms to implement cool things such as face detection, object detection, object classfication, 3D model extraction, and etc.. 

<strong>My OpenCV Experience</strong>

My OpenCV experience is gained from previous [Video Classification Research](https://www.vubblepop.com/vubble-news/vubble-and-seneca-innovative-ai-video-categorization-project?rel=web). In this research project, we need to accomplish video classification with 2 approach. The baseline approach is to use machine learning method. SVM algorithm is mainly involved. The advanced approach is to use deep learning methods. 

OpenCV is a key tool for this research project. We rely it to do feature extraction and model training. OpenCV helps us to extract feature data from thousands of videos and use these data to train the classification model.

<strong>About this project</strong>

In this project I will try to optimize OpenCV. I know it sounds crazy because OpenCV is a big and complex library which is contributed by a lot of experienced developers and CV experts. Althrough I have some experience about it before, my experience is all in application level. It means I only need to know how to use functions/algorithms I need, but never look deeply into its source code, understand the logic of functions/algorithms and how they are implemented. 

For this project, my goal is not ambicious at all and I know I cannot handle that 😅. 

So, I will focus on a very small part of OpenCV, for example, a specific math function and try to see whether I can optimize it with the optimization methods I learned before, for ideally, Aarch64 architecure. 

### <strong>Preparation</strong>

I use this [contribution guidelines](https://github.com/opencv/opencv/wiki/How_to_contribute)  and [Installation in Linux](https://docs.opencv.org/master/d7/d9f/tutorial_linux_install.html) as reference of Installation !!!

<strong>First of all, We need to clone the OpenCV source code</strong>

you can simply use this command
```bash
$ git clone https://github.com/opencv/opencv.git
```

I know that theoritically, If I want to contribute to OpenCV and do some pull request for it, I need to fork it on github first. But at this stage, I just want to get its source code and look around. So I will ignore fork now and do it later. 

<strong>Building OpenCV from source</strong>

Before we start to build, we need to create couple extra directories.

```bash
[wpan17@aarchie project]$ tree -L 1
.
├── build
├── install
├── opencv
└── test
```

This is my struct of the environemnt for OpenCV build and test. 

- build:   where you want to put the generated Makefiles, project files as well the object files and output binaries and enter there.
- install: for CMAKE_INSTALL_PREFIX, define where to install OpenCV, default is "/usr/local"
- opencv:  cloned source code
- test:    contain data and code for testing

The reason why we need to seperate build dir and source dir is that we do not want to mess up our source code. We need to keep our source code in a stable and isolated environemnt. 

After this structure is set up, we can start to build OpenCV with following commends:

```bash
$ cd build
$ cmake CMAKE_INSTALL_PREFIX=../install -S ../opencv/ -B .
$ make -j8
```

Sometimes you might miss some required packages, make tool will complain about it and give you some hints. You can install the missing packages follow its error message. 

### <strong>Searching for opportunities</strong>

Now the building step is done, we can start to browse the source code and find if there is any change we can optimize. 

It gonna take weeks for me to read all source code line-by-line, so here I am going to use a smarter way to search. 

Because OpenCV uses a modular structure, which means that the package includes several shared or static libraries. I am going to focus on the modules directory because ot contains the source files of almost all opencv functions and algorithms.


Here I just use the following commands to search whether OpenCV developer write any arch specific code for various arch.

```bash
$ grep -rnw '.' -e '__x86_64__'
$ grep -rnw '.' -e '__aarch64__'
```

Some interesting things I found is that the in file <strong>"fast_math.hpp"</strong>, it checks conditions for x86_64 couple times but only one time for Aarch64. So, fast_math might have some potential to optimize for aarch64. So let's take a closer look to fast_math!

```
./core/include/opencv2/core/fast_math.hpp:71:      || (defined __GNUC__ && defined __x86_64__ && defined __SSE2__)) \
./core/include/opencv2/core/fast_math.hpp:130:        defined(__x86_64__) || defined(__i686__) \
./core/include/opencv2/core/fast_math.hpp:201:#elif ((defined _MSC_VER && defined _M_X64) || (defined __GNUC__ && defined __x86_64__ \
./core/include/opencv2/core/fast_math.hpp:292:#elif defined(__x86_64__) || defined(_M_X64) || defined(__aarch64__) || defined(__PPC64__)
./core/include/opencv2/core/fast_math.hpp:312:#elif ((defined _MSC_VER && defined _M_X64) || (defined __GNUC__ && defined __x86_64__ \
```

```
./core/include/opencv2/core/fast_math.hpp:292:#elif defined(__x86_64__) || defined(_M_X64) || defined(__aarch64__) || defined(__PPC64__)
```

### <strong>Looking into fast_math.hpp</strong>

After a brief scan of [fast_path.hpp](https://github.com/opencv/opencv/blob/master/modules/core/include/opencv2/core/fast_math.hpp), I realized that this file is trying to use inline way to double/float number rounding for optimization. 

The following code is the code for OpenCV round optimization. We can see that it tries to use special round function if "__GNUC__ && __x86_64__". 
```_mm_cvtss_si32``` is a intel intrinsics and it bascially convert the lower single-precision (32-bit) floating-point element in a to a 32-bit integer, and store the result in destination. At this point, I felt that cvRound should be an cpu-intensive task. Otherwise, openCV developer will not spend time to specifically use intrinstics funtion for x86_64. 

However, I didn't find any special case for __aarch64__.  So firstly I need to find which round method will be used for Aarch64 processor. I add printf function on each condition and re-compile the souce code.  

```c++
CV_INLINE int cvRound(float value)
{
#if defined CV_INLINE_ROUND_FLT
    printf("#if defined CV_INLINE_ROUND_FLT\n");
    CV_INLINE_ROUND_FLT(value);
#elif ((defined _MSC_VER && defined _M_X64) || (defined __GNUC__ && defined __x86_64__ \
    && defined __SSE2__ && !defined __APPLE__) || CV_SSE2) \
    && !defined(__CUDACC__)
    __m128 t = _mm_set_ss( value );
    printf("(defined _MSC_VER && defined _M_X64) || (defined __GNUC__ && defined __x86_64__ \
    && defined __SSE2__ && !defined __APPLE__) || CV_SSE2) LOL\
    && !defined(__CUDACC__)\n");
    return _mm_cvtss_si32(t);
#elif defined _MSC_VER && defined _M_IX86
    int t;
    printf("#elif defined _MSC_VER && defined _M_IX86\n");
    __asm
    {
        fld value;
        fistp t;
    }
    return t;
#elif defined CV_ICC || defined __GNUC__
    printf("#elif defined CV_ICC || defined __GNUC__\n");
    return (int)(lrintf(value));
#else
    /* it's ok if round does not comply with IEEE754 standard;
     the tests should allow +/-1 difference when the tested functions use round */
    printf("return (int)(value + (value >= 0 ? 0.5f : -0.5f));");
    return (int)(value + (value >= 0 ? 0.5f : -0.5f));
#endif
}
```

The result shows that the cvRound go to the ```#elif defined CV_ICC || defined __GNUC__``` and use the ```lrinf(value)``` function. 

### <strong>Benchmark</strong>

In order to test the performance of the current rounding method for Aarch64, I need to create a test and use tools ```grof``` and ```perf``` to see whats going on inside of it.  

Here I created a very simple test case. I create a loop and it will call cvRound(2.5) for 10000000 times. 

```c++
#include <stdio.h>
#include <string.h>
#include <time.h>
#include <opencv2/opencv.hpp>
#include <math.h>
using namespace cv;
using namespace std;

int main(int argc,char** argv){
if( argc != 2 )
    {
#if __aarch64__
printf("__aarch64__\n");
#endif
    for (int i=0;i<10000000;i++){
        cvRound(2.5);
    }
    return 0;}
}
```

After compile, I use ```bash perf record ./main``` to look into it. 

The result for cvRound function is:

 ```bash 
 18.45%  main     main                          [.] cvRound
 ```
If we go inside of cvRound, we can see this result: 
```
       │    0000000000406764 <cvRound(double)>:
       │    _ZL7cvRoundd():
 11.33 │      stp    x29, x30, [sp, #-32]!
 13.12 │      mov    x29, sp
 25.19 │      str    d0, [sp, #24]
 13.40 │      ldr    d0, [sp, #24]
       │    → bl     lrint@plt
 12.48 │      ldp    x29, x30, [sp], #32
 24.48 │    ← ret
```

The cvRound will call another function called "lrint". Its logic of is:
```
       │   0000000000044218 <lrint>:
       │   lrint():
 37.42 │     frintx d0, d0
 50.65 │     fcvtzs x0, d0
 11.93 │   ← ret
```

I did some research about ```lrint``` and I found that it is a very efficient round method for Aarch64. According to the Arm Reference Manual for ArmV8,

```FRINTX Dd, Dn``` will round to integral exact using FPCR rounding mode, single-precision, from Sn to Sd.
```FCVTZSVd.<T>, Vn.<T> ```is Floating-point convert to signed fixed-point of same size (vector) with rounding towards zero. Where <T> is 2S, 4S or 2D. The number of fractional bits is represented by fbits in the range 1 to 64.

These are all new A64 instruction set used in AArch64 state!

At this point, I feel a bit disappointed because it seems cvRound is already optimized enough and there is no room for me do extra stuff. Althrough the developer did not make a special case for Aarch64, the ```lrint()``` is a very efficient and suitable function for it. 

NOOOOOO! It is an awful situation! I already put so much effort to investigate fast_math, I don't want to give up so easily!!!
So, I decided to keep searhing opportunities. 

After I scroll down couple lines, I found that fast_math.cpp also work with other math functions, such as ```cvFloor``` and ```cvCeil```. They seems did not optimized these two functions as much as ```cvRound```. 

For example, ```cvCeil``` function looks much simpler than ```cvRound```. It did not check condition for __86_64__ or others. The only special case is if you are using PowerPC64 and defined to use builtin functions. 


```c++
CV_INLINE int cvCeil( double value )
{
#if (defined CV__FASTMATH_ENABLE_GCC_MATH_BUILTINS || defined CV__FASTMATH_ENABLE_CLANG_MATH_BUILTINS) \
    && ( \
        defined(__PPC64__) \
    )
    return __builtin_ceil(value);
#else
    int i = (int)value;
    return i + (i < value);
#endif
}
```

So I decided to add ```cvCeil``` to my test loop and to see what gonna happen. 

This time ```perf report``` shows me this result:

```
  35.47%  main     main                        [.] cvCeil                      ◆
  13.00%  main     main                        [.] cvRound                     ▒
```

cvCeil and cvRound are using the similar logic and being executed for same amount of times, but the cpu usage for cvCeil is almost triple, compare with cvRound. So, I believe that there should be some room for me to optimize cvCeil and cvFloor.

### <strong>Conclusion for Stage I</strong>
After a long investigation, I understand more about how software optimization work. The openCV optimization looks very professional and smart. It is kinda lucky that I finally find something I can work on. 

The methods I am going to use are either <strong>In-line assembler</strong> or <strong>optimization function</strong> similar with ```lrint```. Althrough ```cvCeil``` and ```cvFloor``` are two simple math functions, there should be a little bit value to make them more efficient, I guess? Since openCV developer already put some effort to optimize the cvRound. If people use openCV to do large amount of calculation(it always happens), use more optimized ```cvCeil``` probably gonna be efficient than use ```int i = (int)value; return i + (i < value);```.