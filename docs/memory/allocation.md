# 内存分配

Go语言的内存分配基于 TCMalloc ，TCMalloc 是Google开发的一个内存分配器。常见的内存分配器还有glic的 PTMalloc 和Google的另一个内存分配器 JEMalloc。
相比于 PTMalloc ，TCMalloc 性能更好，特别适用于高并发场景。
TCMalloc 具有现代化内存分配器的基本特性：对抗内存碎片、在多核处理器上可伸缩。


## 内存管理基本概念

### 内存地址


### 内存中的堆和栈
