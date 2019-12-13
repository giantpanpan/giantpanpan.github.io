---
layout: post
title:  "OpenCV Optimization Practice - Stage III"
date:   2019-12-13 11:44:00
categories: SoftwareOptimization
tags: OpenCV
---
* content
{:toc}

# <strong>OpenCV Optimization - Stage III</strong>

<p><img src="https://giantpanpan.github.io/img/opencv.png" alt="OpenCV"/></p>

Hi everyone, thank you for reading my blog! This is the final stage of my OpenCV Optimization Practice. In this stage, I need to finish the following tasks:
-    Push the optimization to the upstream
-    Conclusion and Review of this project

## <strong>Push To Upstream</strong>
This link is the [OpenCV contribution instruction](https://github.com/opencv/opencv/wiki/How_to_contribute), it will give you step-by-step guide about how to contribute to opencv. 

In Brief, the steps I followed to make a pull-request is
1. Fork opencv
2. Clone forked opencv to my Archie and Betty
3. Create a new branch with meaningful name
4. Modify/add code as we talked about in stage II.
5. Run test locally. We need to glone the [testdate]( https://github.com/opencv/opencv_extra) realsed by opencv. Export the path of the testdata and run each test in build directory. 
6. If the test pass, I can push my new branch to my github fork and create a pull request to opencv base branch. 
7. Testing and merging pull requests

This is a very legit way to create a pull request and is not hard to implement. 

## <strong>Local Test Issues</strong>
In step 6, I found that not all local test passed. For example in ```./opencv_test_core ```:

[==========] 11357 tests from 240 test cases ran. (164504 ms total)
[  PASSED  ] 11354 tests.
[  FAILED  ] 3 tests, listed below:
[  FAILED  ] Core_InputOutput.filestorage_base64_basic_read_XML
[  FAILED  ] Core_InputOutput.filestorage_base64_basic_read_YAML
[  FAILED  ] Core_globbing.accuracy

I failed in 3 locak tests but I have no clue why this happend. I don't think the modification I made can cause these failures. The same situation happened in several other tests too, for example ```./opencv_test_ml```. I guess it is not my fault.

## <strong>Pull Request Status</strong>
Anywait I created a pull Request to Opencv upstream and here is the link [https://github.com/opencv/opencv/pull/16160](https://github.com/opencv/opencv/pull/16160). 

Unfortunately, I found that the build on remote did not pass because of precommit_docs build failed. I checked the build log and I found that the build failure is caused by "modules/core/include/opencv2/core/fast_math.hpp:417: new blank line at EOF.". Such a silly mistake. I fixed this issue and redo they commit. This is the build error log:

```
try to acquire commands lock
commands lock was acquired in 0.000273942947388 sec

git diff --check 5b0b59ecfb1b670fa3a152a8b2b0048d4b128b31 --
 in dir /build/precommit_docs/opencv (timeout 1200 secs)
 watching logfiles {}
 argv: ['git', 'diff', '--check', '5b0b59ecfb1b670fa3a152a8b2b0048d4b128b31', '--']
 environment:
  HOME=/home/build
  OPENCV_GIT_URL=git://code.ocv/opencv
  PATH=/env/bin:/app/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
  PWD=/build/precommit_docs/opencv
  SHELL=/bin/bash
  TERM=xterm
  USER=build
 using PTY: False
modules/core/include/opencv2/core/fast_math.hpp:417: new blank line at EOF.
program finished with exit code 2
elapsedTime=0.029254


commands lock was released
```

 Waiting for building:
<p><img src="https://giantpanpan.github.io/img/waiting_for_build.png" alt="OpenCV"/></p>


## <strong>Overview of OpenCV Optimization Practice<strong>
Overall, this is a very meaning project to me. The most important thing I learned from this project is the methodology of 
1. <strong>How to find a software optimization opportunity</strong>
2. <strong>How to make a good optimization plan for the opportunity you found</strong>
3. <strong>How to verify whether your optimization plan works</strong> 

If OSD600 (opensource development) is the course lead me to the open source world, SPO600 gives me the chance to dig into some large open-source software project and learn how to implement a proper software optimization.   

