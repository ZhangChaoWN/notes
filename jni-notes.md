## 工具的安装与使用

JNI 开发过程中，把 C/C++ 编写的 native lib 编译到不同目标平台，需要使用到编译器、链接器等工具。

### linux
使用gcc/g++，ubuntu linux 中通过包管理器一行命令即可安装好所需的编译工具。具体的命令没有记录下来，搜索引擎可以很方便地搜到。

### macOS
使用 clang 编译器，mac开发者命令行工具包中包含了clang，开发者不需要刻意地去安装工具包，直接使用这些工具即可，如果操作系统发现当前没有安装工具包，会弹出一个弹窗，询问是否自动下载并安装工具包中的工具。工具包除clang外也包含了一些其它开发者常用的命令行工具。

### windows
使用mingw-w64 交叉编译环境中的gcc/g++，安装方法：按 [msys2 官网](https://www.msys2.org/)安装说明安装 msys2，再通过命令：`pacman -S mingw-w64-gcc` 在msys2中安装mingw-w64-gcc

其它工具试用记录：
一开始使用 mingw 作为开发环境，发现 mingw 环境只能编译到32bit目标平台，32bit 动态库无法被 64bit JVM 加载（Oracle HotSpot Jvm 1.8）。后来发现还有另外一个项目，叫做 mingw-w64，从名字上可以看出这个项目与 mingw 有历史上的关系。相关背景知识：[[科普][FAQ]MinGW vs MinGW-W64及其它](https://github.com/FrankHB/pl-docs/blob/master/zh-CN/mingw-vs-mingw-v64.md) 

## 问题
- 为了编译到不同操作系统，准备多台机器（虚拟机）不方便，是否可以通过交叉编译，在一个操作系统中编译出多个目标平台的程序？

- JNI 文档中有一个看起来比较特殊的叫做 UTF8 String 字符串数据结构，这个数据结构具体是如何编码字符串的？这个数据结构与 C/C++ 中的其它字符串有何异同，是否兼容？

## 一些文档摘录

> As of Java SE 6.0, the deprecated structures JDK1_1InitArgs and JDK1_1AttachArgs have been removed, instead JavaVMInitArgs and JavaVMAttachArgs are to be used.

相对Java SE 6.0, 唯一修改是删除了两个已弃用结构体

> Native method programmers should program to the JNI. Programming to the JNI insulates you from unknowns, such as the vendor’s VM that the end user might be running. By conforming to the JNI standard, you will give a native library the best chance to run in a given Java VM.

> Binary compatibility - The primary goal is binary compatibility of native method libraries across all Java VM implementations on a given platform. Programmers should maintain only one version of their native method libraries for a given platform.

以上内容来源：https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/intro.html



## 其它资料

https://docs.oracle.com/javase/7/docs/technotes/guides/jni/index.html
https://docs.oracle.com/javase/8/docs/technotes/guides/jni/index.html
https://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/design.html