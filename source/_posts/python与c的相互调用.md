---
title: python 与 c 的相互调用
tags:
  - python
  - c
date: 2016-11-17 22:43:34
---

   python写出的程序被称为‘能运行的伪代码’， 因为它语法简单，能极大的解放程序员的思想，不需要使用者太关注细节，但是它效率较低，处理大量数据时需要大量时间。而C语言则正好相反，接近底层，运行速度快，但是使用时要处处小心，要不然轻则编译不通过，重则运行时崩溃。

   若能将两者结合起来，岂不是妙哉？

<!--more-->

## python 调用 c 程序

1.  首先我们编写一个简单的 c 程序 `multi.c`

     ```c
     #include <stdio.h>
     int multiply(int n1, int n2)
     {
         return n1 * n2;
     }
     ```

2. 接着将写好的 c 程序编译生成动态库 `multi.so`，便于 python 程序调用。
   `gcc -shared -fPIC multi.c -o multi.so`

   *   -shared  生成动态库
   *   -fPIC 生成与位置无关代码(Position-Independent Code)
   *   -o 指定生成库的名字

3. 最后在 `multi.so` 的同级目录下运行 python 解释器, 调用刚刚编写的 multiply 函数

     ```python
     >>> import ctypes
     >>> multi = ctypes.cdll.LoadLibrary('./multi.so')
     >>> multi.multiply(2, 4)
     8
     ```

`ctypes` 库提供了与c语言一致的数据类型，并且可以用来调用动态链接库中的函数。在这里，我们通过`ctypes.cdll.LoadLibrary`来读取c库。该库更详细的使用可以参阅[官方文档 ctypes — A foreign function library for Python](https://docs.python.org/2/library/ctypes.html)