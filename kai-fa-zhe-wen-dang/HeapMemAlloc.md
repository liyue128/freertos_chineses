# 堆栈内存管理

每次创建任务、队列、互斥锁、软件定时器、信号量或事件组时，RTOS内核都需要使用RAM。RAM可以在RTOS API对象创建函数从RTOS堆动态分配，也可以由应用程序开发者指定分配。

如果RTOS对象是动态创建的，那么有时可以使用标准C库malloc()和free()函数来实现，但是...

1.   这些函数并不总是在嵌入式系统上可用；
2.   占用了本就不多的代码空间；
3.   在线程上并不一定是安全的；
4.   不是确定性的（执行函数所花费的时间会因调用而异）；

...因此通常需要另一种内存分配方案。

一个嵌入式/实时系统可能具有与另一个系统具有完全不同的RAM和时序要求 - 因此单个RAM分配算法将只适用于某一类应用程序。

为解决该问题，FreeRTOS将内存分配API保留在其可移植层中。可移植层位于RTOS核心文件之外，能够提供适用于特定于应用程序的代码实现。当RTOS内核需要RAM时，不会调用malloc()，而是调用pvPortMalloc()。当RAM被释放时，RTOS内核不会调用free()，而是调用vPortFree()。

FreeRTOS提供了多种堆管理方案，其复杂性和功能各不相同。也可以在应用程序中使用自己的堆实现，甚至可以同时使用两个堆实现。同时使用两个堆实现能将任务堆栈和其他RTOS对象放置在处理速率更快的内部RAM中，将应用程序数据放置在较慢的外部RAM中。

## RTOS源码中的内存分配方案

FreeRTOS源码包括五个示例内存分配方案，每个都在以下小节中进行了描述。 这些小节还包括有关何时选择最合适方案的信息。

每个方案都包含在一个单独的源文件中(分别为heap_1.c、heap_2.c、heap_3.c、heap_4.c和heap_5.c)，该源文件位于主RTOS源代码下的Source/Portable/MemMang目录中。可以根据需要添加其他实现。特定的内存方案在一个项目中只能出现一次(即便应用程序选择使用开发者自己设计的堆实现，但由这些可移植层函数定义的堆仍将会被RTOS内核使用)。

如下所示:

*   [heap_1] - 最简单的方案，不允许释放内存; 
*   [heap_2] - 允许释放内存，但不合并相邻的空闲块。
*   [heap_3] - 简单地封装标准malloc()和free()以确保线程安全。
*   [heap_4] - 合并相邻的空闲块以避免碎片化， 提供绝对地址放置选项。
*   [heap_5] - 能够使用多个非相邻内存区域的堆。

注意: 

*   heap_1适用性较差，因为FreeRTOS支持静态分配。
*   heap_2可认为是旧版本遗留的，因为较新的heap_4实现是应用程序开发的首选。

## heap_1

heap_1是所有方案中最简单的。一旦分配了内存，该方案不允许释放内存(通过static关键字实现)。尽管如此，heap_1.c仍适用于嵌入式应用程序。这是因为许多小型且深度嵌入的应用程序会在系统启动时创建所有所需的任务、队列、信号量等，然后在程序的生命周期内使用但不会删除所有这些对象（直到应用程序再次关闭或重新启动）。

该方案在请求RAM时将单个存储阵列简单地分为更小的块。阵列总大小(堆的总大小)由configTOTAL_HEAP_SIZE设置-在FreeRTOSConfig.h中定义。FreeRTOSConfig.h定义的configAPPLICATION_ALLOCATED_HEAP配置常量以允许将堆放置在内存中的特定地址。

xPortGetFreeHeapSize() API函数返回未分配的堆空间总量，允许对configTOTAL_HEAP_SIZE进行优化。

heap_1方案特点如下:

*   如果应用程序从不删除任务、队列、信号量、互斥锁等(实际上涵盖了大多数使用FreeRTOS的应用程序)，则可以使用。
*   总是确定性的（总是花费相同的时间来执行）并且不会导致内存碎片。
*   实现起来非常简单，从静态分配的数组分配内存，意味着通常适用于不允许真正动态内存分配的应用程序。

## heap_2

heap_2使用最佳拟合算法。与heap_1不同，该方案允许释放先前分配的块，但不会将相邻的空闲块合并为一个大块。

可用堆空间的总量由configTOTAL_HEAP_SIZE决定 - 在FreeRTOSConfig.h中定义。FreeRTOSConfig.h定义的configAPPLICATION_ALLOCATED_HEAP配置常量以允许将堆放置在内存中的特定地址。

xPortGetFreeHeapSize() API函数返回未分配的堆空间总量(允许对configTOTAL_HEAP_SIZE进行优化)，但不提供有关如何将未分配的内存分成更小的块的信息。

heap_2方案特点如下:

*   即使在应用程序重复删除任务、队列、信号量、互斥体等时也可以使用，但是会出现内存碎片。
*   如果分配和释放的内存是随机大小，则不应使用。例如：
    *   如果应用程序动态创建和删除任务，并且分配给正在创建的任务的堆栈大小始终相同，那么在大多数情况下可以使用heap2.c。 但是，如果分配给正在创建的任务的堆栈大小并不总是相同，那么可用的空闲内存可能会碎片化为许多小块，最终导致分配失败。 在这种情况下，heap_4.c将是更好的选择。
    *   如果应用程序动态创建和删除队列，并且每种情况下队列存储区都相同（队列存储区是队列项大小乘以队列长度），那么大多数情况下可以使用heap_2.c。 但是，如果每种情况下的队列存储区域都不相同，那么可用的空闲内存可能会分成许多小块，最终导致分配失败。 在这种情况下，heap_4.c 将是更好的选择。
    *   该应用程序直接调用pvPortMalloc()和vPortFree()，而不仅仅是通过其他FreeRTOS API函数间接调用。
*   如果应用程序的队列、任务、信号量、互斥体等顺序不可预知，可能会导致内存碎片问题。 这对于几乎所有应用程序来说都是不太可能的，但应该牢记在心。
*   具有未知确定性 - 但比大多数标准C库malloc实现更有效。

heap_2.c适用于许多必须动态创建对象的小型实时系统。 有关将空闲内存块组合成单个更大块的类似实现，请参见 heap_4。

## heap_4

该方案使用首次拟合算法，并且与方案 2 不同，能够将相邻的空闲内存块合成一个大块(该方案包含合并算法)。

可用的堆空间由在FreeRTOSConfig.h中定义的configTOTAL_HEAP_SIZE设置。 FreeRTOSConfig.h中定义的configAPPLICATION_ALLOCATED_HEAP能够配置常量以允许将堆放置在内存中的指定地址。

xPortGetFreeHeapSize() API函数返回调用该函数时未分配的堆空间总量，而xPortGetMinimumEverFreeHeapSize() API函数返回FreeRTOS应用程序启动时已存在系统的最小可用堆空间量。 这两个函数都没有提供有关如何将未分配的内存分成更小的块的信息。

vPortGetHeapStats() API函数提供附加信息，填充了heap_t结构的成员，如下所示

```cpp
/* Prototype of the vPortGetHeapStats() function. */
void vPortGetHeapStats( HeapStats_t *xHeapStats );

/* Definition of the Heap_stats_t structure. */
typedef struct xHeapStats
{
	size_t xAvailableHeapSpaceInBytes;      /* The total heap size currently available - this is the sum of all the free blocks, not the largest block that can be allocated. */
	size_t xSizeOfLargestFreeBlockInBytes; 	/* The maximum size, in bytes, of all the free blocks within the heap at the time vPortGetHeapStats() is called. */
	size_t xSizeOfSmallestFreeBlockInBytes; /* The minimum size, in bytes, of all the free blocks within the heap at the time vPortGetHeapStats() is called. */
	size_t xNumberOfFreeBlocks;		/* The number of free memory blocks within the heap at the time vPortGetHeapStats() is called. */
	size_t xMinimumEverFreeBytesRemaining;	/* The minimum amount of total free memory (sum of all free blocks) there has been in the heap since the system booted. */
	size_t xNumberOfSuccessfulAllocations;	/* The number of calls to pvPortMalloc() that have returned a valid memory block. */
	size_t xNumberOfSuccessfulFrees;	/* The number of calls to vPortFree() that has successfully freed a block of memory. */
} HeapStats_t;
```

heap_4方案特点如下:

*   在应用程序重复删除任务、队列、信号量、互斥体等时也可使用;
*   与heap_2实现相比，导致堆空间严重碎片化为多个小块的可能性要小得多--即使分配和释放的内存大小是随机的。
*   具有不确定性 - 但比大多数标准C库malloc实现更有效

heap_4.c对想要在应用程序代码中直接使用可移植层内存分配方案的应用程序特别有用(而不是通过调用pvPortMalloc()和vPortFree()的API函数间接使用)。

## heap_5

该方案使用与heap_4相同的首次拟合和内存合并算法，并允许堆跨越多个非相邻（非连续）内存区域。

Heap_5通过调用vPortDefineHeapRegions()完成初始化，并且在vPortDefineHeapRegions()执行之后才能使用。创建RTOS对象（任务、队列、信号量等）将隐式调用pvPortMalloc()，因此在使用heap_5时，必须在创建任何此类对象之前调用vPortDefineHeapRegions()。

vPortDefineHeapRegions()函数只有一个参数。 该参数是在portable.h 中定义的HeapRegion_t结构数组: 

```cpp
typedef struct HeapRegion
{
    /* Start address of a block of memory that will be part of the heap.*/
    uint8_t *pucStartAddress;

    /* Size of the block of memory. */
    size_t xSizeInBytes;
} HeapRegion_t;

The HeapRegion_t type definition
```

NULL用来终止数组，数组中定义的内存区域必须按地址顺序出现，从低地址到高地址。以下源代码片段提供了一个示例。MSVC Win32 模拟例程也使用heap_5，可用作参考:

```cpp
/* Allocate two blocks of RAM for use by the heap.  The first is a block of
0x10000 bytes starting from address 0x80000000, and the second a block of
0xa0000 bytes starting from address 0x90000000.  The block starting at
0x80000000 has the lower start address so appears in the array fist. */
const HeapRegion_t xHeapRegions[] =
{
    { ( uint8_t * ) 0x80000000UL, 0x10000 },
    { ( uint8_t * ) 0x90000000UL, 0xa0000 },
    { NULL, 0 } /* Terminates the array. */
};

/* Pass the array into vPortDefineHeapRegions(). */
vPortDefineHeapRegions( xHeapRegions );

Initialising heap_5 after defining the memory blocks to be used by the heap
```

xPortGetFreeHeapSize() API函数返回调用该函数时未分配的堆空间总量，而xPortGetMinimumEverFreeHeapSize() API函数返回FreeRTOS应用程序启动时已存在系统的最小可用堆空间量。这两个函数都没有提供有关如何将未分配的内存分成更小的块的信息。

vPortGetHeapStats() API函数提供有关堆状态的附加信息。
