---
layout: post
title:  "Software Optimization - Building Software"
date:   2019-11-10 13:11:00
categories: SoftwareOptimization
tags: buildSoftware
---
* content
{:toc}

## <strong>Building Software</strong>
This week we are going to discuss how to build a software. I need to learn more about it because I am going to try optimizing an open source package/library. In order to work with it, at least I need to build it successfully firstly. So let us take a look about the modern software building process. 




Basically, if you want to develop a package which can be used by others, you need a build system. If you just want to build a very simple package and do not want to consider issues like cross platform, you can create a Makefile and define how you want to compile your software there. 

### <strong>Make</strong> 
All you need to do is use command ```$ make``` then you will get a executable file and couple object tiles. 

However, if you want to build your package on a different platform, we need to change the content of the Makefile. In this case, we better to create a configuration file. 

Here is my example of CMakeList.txt to build an executable called ```main```. It requires openCV library. 

```
cmake_minimum_required(VERSION 2.8)
project( main )
find_package( OpenCV REQUIRED )
add_executable( main main.cpp )
target_link_libraries( main ${OpenCV_LIBS} )
```
### <strong>Cmake</strong>
You need to use command like ```$ cmake``` to generate a Makefile automatically. 

After that, you can use ```$ make -j8``` to compile the package using 8 parallel process. 

Before we start to build, we need to create couple extra directories, such as build directory. The reason why we need to seperate build dir and source dir is that we do not want to mess up our source code. We need to keep our source code in a stable and isolated environemnt. 

### <strong>Compiler Options</strong>
There are also couple parameters we can use for building packages.

```PATH=".:$PATH"```: set the path to executable on a particular path

```LD_LIBRARY_PATH```: Specify the particular lib path to search first, otherwise, it will use default path.