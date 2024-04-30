---
title: 一个“size_type”引发的Bug
date: 2024-04-25 15:29:57
tags: 
- 原创
- ELF
- gcc
- 共享目标文件
- 链接
categories:
- 踩坑日记
---

# ELF 可执行文件被识别为共享目标文件

## 问题描述

个人项目中有一个解析 ELF 文件类型的功能，在测试过程中，错误地将可执行文件 (ET_EXEC) 识别为共享目标文件 (ET_DYN)。

如下图所示，exec_test 必然是一个可执行文件：

![](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/notebooks/uploads/43dad8fbc3f5c70f79ab0c2d3724eefe/2024-04-25_155005.png)

但是当我无论使用 `readelf -h` 查看 exec_test 的头部信息，还是使用 `file` 命令查看文件类型，都显示这是一个共享目标文件：

![](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/notebooks/uploads/371fe11454fc9e9404b2ee5e147c24a7/2024-04-25_155806.png)

通过上面第一张图片中的编译指令，可以看到仅仅使用了最常规的 `-g -O0 -o` 选项，并没有使用额外的编译或者链接选项。

## 系统环境

+ OS: Ubuntu 20.04.6 LTS x86_64
+ Compiler: g++ (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0

## ELF 文件简单介绍

一般来说，ELF 文件有 4 种：

+ 可重定位文件 (ET_REL)
+ 可执行文件 (ET_EXEC)
+ 共享目标文件 (ET_DYN)
+ 核心转储文件 (ET_CORE)

| 文件类型 | 描述 |
| :--- | :--- |
| 可重定位文件 (ET_REL) |<li>待重定位文件是由编译器产生的中间文件，包含了程序的机器代码、符号表、重定位信息等。</li><li>它是源代码编译后生成的目标文件，但还没有链接成可执行文件或共享目标文件。</li><li>待重定位文件中的地址信息仍然是相对地址，需要在链接时进行地址重定位。</li>|
| 可执行文件 (ET_EXEC) |<li>可执行文件包含了可以直接在操作系统上运行的程序代码和数据。</li><li>它可以被操作系统加载到内存中，并执行其中的指令。</li><li>在Unix/Linux系统中，可执行文件通常没有文件扩展名，但是在Windows系统中通常使用 .exe 扩展名。</li>|
| 共享目标文件 (ET_DYN) |<li>共享目标文件是包含了可重用代码和数据的文件，可以被多个可执行文件动态链接和共享使用。</li><li>它以一种与操作系统和其他程序共享的形式存在，可以在程序运行时动态加载到内存中。</li><li>共享目标文件通常具有文件扩展名 .so（Unix/Linux）或 .dll（Windows）。</li>|
| 核心转储文件 (ET_CORE) |<li>Core 文件是在程序崩溃或异常退出时由操作系统自动生成的一种文件。</li><li>它包含了程序崩溃时的内存快照信息，包括堆栈信息、寄存器状态等。</li><li>Core 文件通常用于调试程序崩溃的原因，可以通过调试工具分析其内容以定位问题。</li>|

也可见下图：

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bab723b24d954cd39cf89370b95a1cdf~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

## 问题原因

在查找了资料后，得知是由于高版本 gcc/g++ 默认使用 `-pie` 选项，导致生成的可执行文件被识别为共享目标文件。

`-pie` 选项是 Position Independent Executable 的缩写，即生成位置无关可执行文件。这种可执行文件可以被加载到任意地址运行，而不需要进行地址重定位。这样可以提高程序的安全性，防止恶意程序利用地址重定位漏洞进行攻击。

## 解决方案

解决方法有两种：

1. 使用 `-no-pie` 选项，禁用生成位置无关可执行文件：

```shell
g++ -g -O0 -no-pie -o exec_test test.cc
```

2. 使用 `-fno-pie` 选项，禁用生成位置无关可执行文件：

```shell
g++ -g -O0 -fno-pie -o exec_test test.cc
```

![](https://gitlab.b1gfac3c4t.top:1594/xu4nch3n/notebooks/uploads/d79f24ba7a2a8e259a4011aad40cafb8/2024-04-25_163212.png)

可以看到，在添加了 `-no-pie` 选项后，exec_test 被正确识别为可执行文件。
