# java/JVM
**问题**：Jvm启动时通过Xms设置初始堆大小，JVM启动后是否真的会使用这么多宿主机内存


stackoverflow 上的一个问答： [Do the -Xms and-Xmx flags reserve the machine's resources?](https://stackoverflow.com/questions/43302720/) 中给出了这样的解释：

> Short answer: Depends on the OS, though it's definitely a NO in all popular operating systems.

> I'll take the example of Linux's memory allocation terminology here.

> -Xms and -Xmx specify the minimum and maximum size of JVM heap. These sizes reflect VIRTUAL MEMORY allocations which can be physical mapped to pages in RAM called the RESIDENT SIZE of the process at any time.

> When the JVM starts, it'll allocate -Xms amount of virtual memory. This can be mapped to resident memory (physical memory) once you dynamically create more objects on heap. This operation will not require JVM requesting any new allocation from the OS, but will increase you RAM utilization, because those virtual pages will now actually have corresponding physical memory allocation too. However, once your process tries to create more objects on heap after consuming all its Xms allocation on RAM, it has to request the OS for more virtual memory from the OS, which may/may not also be mapped to physical memory later depending on when you need it. The limit for this is your -Xmx allocation.

> Note that this is all possible because the memory in linux is shared. So, even if a process allocates memory beforehand, what it gets is virtual memory which is just an addressable contiguous fictional allocation that may or may not be mapped to real physical pages depending on the demand. Read this answer for a short description of how memory management works in popular operating systems. Here is a much detailed (slightly outdated but very useful) information on how Linux's memory management works.

linux文档关于Anonymous Memory的[解释](https://www.kernel.org/doc/html/latest/admin-guide/mm/concepts.html#anonymous-memory)

> The anonymous memory or anonymous mappings represent memory that is not backed by a filesystem. Such mappings are implicitly created for program’s stack and heap or by explicit calls to mmap(2) system call. Usually, the anonymous mappings only define virtual memory areas that the program is allowed to access. The read accesses will result in creation of a page table entry that references a special physical page filled with zeroes. When the program performs a write, a regular physical page will be allocated to hold the written data. The page will be marked dirty and if the kernel decides to repurpose it, the dirty page will be swapped out.

根据上面内容，我的理解是，在Linux操作系统中程序使用内存分两步：第一步是申请内存空间，第二步是向内存空间写数据。程序向操作系统申请内存得到的是virtual memory，当程序真正向virtual memory写一段数据，数据会被写入resident memory (physical memory)，这段数据占用空间才会被真正计入程序的物理内存占用。

**如何查看程序占用的物理内存？**

resident memory 可以通过top命令查看RES, ps aux命令查看RSS，甚至可以通过文件系统路径`/proc/$(pidof process)/status`查看。[linux RSS from ps RES from TOP](https://stackoverflow.com/questions/15907807/linux-rss-from-ps-res-from-top)

下面两段内容是top命令输出的RES字段的[描述](http://man7.org/linux/man-pages/man1/top.1.html)：
```
anything occupying physical memory which, beginning with
Linux-4.5, is the sum of the following three fields:
RSan - quadrant 1 pages, which include any
    former quadrant 3 pages if modified
RSfd - quadrant 3 and quadrant 4 pages
RSsh - quadrant 2 pages
```

```
22. RES  --  Resident Memory Size (KiB)
    A subset of the virtual address space (VIRT) representing the
    non-swapped physical memory a task is currently using.  It is
    also the sum of the RSan, RSfd and RSsh fields.

    It can include private anonymous pages, private pages mapped to
    files (including program images and shared libraries) plus shared
    anonymous pages.  All such memory is backed by the swap file
    represented separately under SWAP.

    Lastly, this field may also include shared file-backed pages
    which, when modified, act as a dedicated swap file and thus will
    never impact SWAP.

    See `OVERVIEW, Linux Memory Types' for additional details.
```





通过命令行工具 jcmd <pid> VM.native_memory scale=MB 输出的结果中，内存占用会分为malloc和mmap两种。mmap又分为reserved和commited两个取值。

malloc函数定义在头文件`<stdlib.h>`中。mmap定义在`<sys/mman.h>`中。

`sys/mamn.h`遵循什么规范？

malloc是C语言标准函数中用于申请内存的函数。

那么mmap呢？

> 存储映射I/O（memory-mappedI/O）能将一个磁盘文件映射到存储空间中的一个缓冲区上，于是，当从缓冲区中取数据时，就相当于读文件中的相应字节。与此类似，将数据存入缓冲区时，相应字节就自动写入文件。这样，就可以在不使用read和write的情况下执行I/O。

W. Richard Stevens; Stephen A. Rago. UNIX环境高级编程（第3版）（异步图书） (Kindle 位置 9678-9680). 人民邮电出版社. Kindle 版本.

> 为了使用这种功能，应首先告诉内核将一个给定的文件映射到一个存储区域中。这是由mmap函数实现的。

W. Richard Stevens; Stephen A. Rago. UNIX环境高级编程（第3版）（异步图书） (Kindle 位置 9685-9686). 人民邮电出版社. Kindle 版本.

如上两段话是mmap作为操作系统IO函数的介绍，介绍了通过mmap函数把文件系统中的文件映射到虚拟内存用法。mmap还有另一种用法。调用 mmap 库函数时可以通过 MAP_ANONYMOUS标记，请求匿名内存，linux [手册](http://man7.org/linux/man-pages/man2/mmap.2.html)中的说明：
```
MAP_ANONYMOUS
        The mapping is not backed by any file; its contents are
        initialized to zero.  The fd argument is ignored; however,
        some implementations require fd to be -1 if MAP_ANONYMOUS (or
        MAP_ANON) is specified, and portable applications should
        ensure this.  The offset argument should be zero.  The use of
        MAP_ANONYMOUS in conjunction with MAP_SHARED is supported on
        Linux only since kernel 2.4.
```

有趣的一个词汇[ORed是什么意思](https://english.stackexchange.com/questions/79659/the-verb-for-carrying-out-a-bitwise-or-and-operation)


拓展问题：jvm内存管理用到了heap这个词，操作系统中的动态内存分配例如linux malloc好像也用到了heap这个概念。它们是一个意思吗？反正应该都不是数据结构的意思。
拓展阅读：
- https://www.linuxjournal.com/article/6390
- https://en.wikipedia.org/wiki/Heap_(programming)

reserved和committed分别是什么意思？

oracle网站reserved与commmited的[说明](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html)

> Interpret sample output: From the sample output below, you will see reserved and committed memory. Note that only committed memory is actually used. For example, if you run with -Xms100m -Xmx1000m, the JVM will reserve 1000 MB for the Java Heap. Since the initial heap size is only 100 MB, only 100MB will be committed to begin with. For a 64-bit machine where address space is almost unlimited, there is no problem if a JVM reserves a lot of memory. The problem arises if more and more memory gets committed, which may lead to swapping or native OOM situations.



https://gceasy.io/ 的诊断报告中：

> Note: there are 8 flavours of OutOfMemoryErrors. With GC Logs you can diagnose only 5 flavours of them(Java heap space, GC overhead limit exceeded, Requested array size exceeds VM limit, Permgen space, Metaspace). So in other words, your application could be still suffering from memory leaks, but need other tools to diagnose them, not just GC Logs.

gceasy.io给出的文档链接：[outofmemoryerror2.pdf](https://tier1app.files.wordpress.com/2014/12/outofmemoryerror2.pdf)


[jvm参数解释](https://stackoverflow.com/questions/10486375/print-all-jvm-flags)

## 工具
oracle的java troubleshooting guide建议使用jcmd替代jstack，原文：
> The release of JDK 8 introduced Java Mission Control, Java Flight Recorder, and jcmd utility for diagnosing problems with JVM and Java applications. It is suggested to use the latest utility, jcmd instead of the previous jstack utility for enhanced diagnostics and reduced performance overhead.
来源：https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr016.html

## 线程的状态

问题的背景：java程序不响应的情况下，可以通过jstack工具列出程序中的线程的栈信息，通过逐个查看线程的栈，可以得出大多数线程都在执行至哪行代码的时候block住了，例如等待HTTP应答并且设置了很长的超时时间，导致线程池中没有线程可用，所以服务器无法响应新的请求，这个分析方法主要关注的是栈信息，也就是哪个方法调了哪个方法，线程当前执行到了哪一行代码，但是jstack输出中还有一个线程状态信息不知道什么意思（例：java.lang.Thread.State: RUNNABLE），如果能弄清楚，对于以后的问题排查分析或许会有帮助。

可以考虑查阅这些资料：
jstack工具的文档
jvm specification
java doc java.lang.Thread
搜索引擎搜索thread state

java doc：https://docs.oracle.com/javase/7/docs/api/java/lang/Thread.State.html 中有简要的说明，但是有一些细节还是不是非常清晰。

## 如何解读 cms gc log
https://blogs.oracle.com/poonam/understanding-cms-gc-logs
