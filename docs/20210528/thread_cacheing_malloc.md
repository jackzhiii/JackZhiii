# TCMalloc: Thread-Caching Malloc

## Motivation
TCMalloc 是一个内存分配器，旨在替代系统默认的分配器。它有以下的特性：
1. 快速：对大部分对象而言，无竞争分配和释放。根据模式（每个线程或者每个逻辑CPU）来缓存对象。大多数分配都不需要加锁，所以只有很少的竞争，这有利于多线程应用的扩展。

2. 灵活使用内存，所以空闲内存可以被各种尺寸的对象重复使用，或者还给操作系统。

3. 通过分配相同大小对象的 “pages”, 降低了每个对象的内存开销。使小物体的标示具有空间效率。

## Usage 
通过在 Bazel 中指定 TCMalloc 作为二进制的 malloc 属性来使用 TCMalloc

## Overview
下面的框图展示了 TCMalloc 粗糙的内部架构
![](../../images/tcmalloc_internals.png)

我们可以将 TCMalloc 分为三个组件。前端， 中间端，后端。我们将会在下面的部分进行更加详细的讨论。一个粗略的责任分离如下：

1. 前端：前端是一个缓存，用于快速的对应用内存的分配和回收

2. 中端：负责前端缓存的填充

3. 后端用于处理从操作系统中获取内存

注意前端可以在 per-CPU 或者 per-thread 模式下运行，后端可以支持 hugepage aware pageheap 或者 legacy pageheap。

## TCMalloc front-end
前端处理特定大小的内存请求。前端拥有内存的缓存，它可以用来分配或维持空闲内存。这些缓存在同一时间只能被同一个缓存获取，所以这个过程中并不需要任何锁，因此大部分分配和回收都非常快。

前端将会让任何请求感到满意当它已经缓存了何时寸尺的内存。如果 cache 中没有合适尺寸的内存，那么 front-end 将会从 middle-end 请求批量的内存来重新填充。middle-end 由 CentralFreeList 和 TransferCache 组成。

如果 middle-end 耗尽，或者请求内存的大小大于 front-end 混存的最大大小，则请求将会转到后端以满足大型分配，或者重新填充 middle-end 的缓存。back-end 同样被称为 PageHeap。

这有两种方式实现了 TCMalloc front-end:

1. 原生的 TCMalloc 支持 per-thread 对象缓存（名字由此而来）。但是，这导致内存占用量与线程数量成比例增长。现代应用程序可以拥有很大的线程数，这将导致要么每个线程分配的内存总计非常大，或者导致每个线程只有极小的缓存。

2. 最近 TCMalloc 已经支持了 per-CPU 模式。在这种模式下，系统中每个逻辑的CPU都有它自己的缓存用于分配内存。注意：在 x86 上，一个逻辑 CPU等效于超线程。

per-thread 和 per-CPU 模式之间的区别完全局限于 malloc/new 和 free/delete 的实现。

## Small and Large Object Allocation
"small" 对象的分配被映射到