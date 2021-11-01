# 静态和动态内存分配

## 概述

FreeRTOS V9.0.0之前的版本从指定的FreeRTOS堆分配下面列出的RTOS对象使用的内存。FreeRTOS V9.0.0及更高版本使应用程序能够使用指定内存，从而允许在不动态分配任何内存的情况下选择性地创建以下对象：

*   任务
*   软件定时器
*   队列
*   事件组
*   二进制信号量
*   计数信号量
*   递归信号量
*   互斥锁

使用静态内存分配还是动态内存分配取决于应用程序的具体使用场景。两种方法各有利弊，并且两种方法都可以在同一个RTOS应用程序中使用。

## 使用动态分配的RAM创建RTOS对象

动态创建RTOS对象的好处是更加简单，并且有可能最大限度地减少应用程序的RAM使用量：

*   创建对象时需要的函数参数较少;
*   内存分配在RTOS API函数内自动发生;
*   应用程序开发者不需要关心分配内存;
*   如果对象被删除，RTOS对象使用的RAM可重新使用，减少应用程序的RAM占用空间;
*   提供RTOS API函数来返回有关堆使用情况的信息，从而优化堆大小;
*   可以选择最适合应用程序的内存分配方案，heap_1.c用于安全关键应用程序所需要的简单性和确定性，heap_4.c用于碎片保护，heap_5.c用于将堆拆分到多个RAM区域，或应用程序提供的分配方案。

以下API函数在configSUPPORT_DYNAMIC_ALLOCATION设置为1或未定义时可用，使用动态分配的RAM创建RTOS对象：

*   [xTaskCreate()](https://freertos.org/a00125.html)
*   [xQueueCreate()](https://freertos.org/a00116.html)
*   [xTimerCreate()](https://freertos.org/FreeRTOS-timers-xTimerCreate.html)
*   [xEventGroupCreate()](https://freertos.org/xEventGroupCreate.html)
*   [xSemaphoreCreateBinary()](https://freertos.org/xSemaphoreCreateBinary.html)
*   [xSemaphoreCreateCounting()](https://freertos.org/CreateCounting.html)
*   [xSemaphoreCreateMutex()](https://freertos.org/CreateMutex.html)
*   [xSemaphoreCreateRecursiveMutex()](https://freertos.org/xSemaphoreCreateRecursiveMutex.html)

## 使用静态分配的RAM创建RTOS对象

使用静态分配的RAM创建RTOS对象能够利于应用程序开发者的工作：

*   RTOS对象可以放置在指定的内存位置;
*   可以在链接时而不是运行时确定最大RAM占用空间;
*   应用程序开发者不需要关心如何处理内存分配失败的情况;
*   允许RTOS用于不允许动态内存分配的应用程序。

以下API函数在configSUPPORT_STATIC_ALLOCATION设置为1时可用，允许使用应用程序开发者指定的内存创建RTOS对象。为创建内存，应用程序编写者只需声明一个适当对象类型的变量，然后将变量的地址传递给RTOS API函数。StaticAllocation.c提供标准演示/测试任务来演示如何使用这些函数：

* [xTaskCreateStatic()](https://freertos.org/xTaskCreateStatic.html)
* [xQueueCreateStatic()](https://freertos.org/xQueueCreateStatic.html)
* [xTimerCreateStatic()](https://freertos.org/xTimerCreateStatic.html)
* [xEventGroupCreateStatic()](https://freertos.org/xEventGroupCreateStatic.html)
* [xSemaphoreCreateBinaryStatic()](https://freertos.org/xSemaphoreCreateBinaryStatic.html)
* [xSemaphoreCreateCountingStatic()](https://freertos.org/xSemaphoreCreateCountingStatic.html)
* [xSemaphoreCreateMutexStatic()](https://freertos.org/xSemaphoreCreateMutexStatic.html)
* [xSemaphoreCreateRecursiveMutexStatic()](https://freertos.org/xSemaphoreCreateRecursiveMutexStatic.html)
