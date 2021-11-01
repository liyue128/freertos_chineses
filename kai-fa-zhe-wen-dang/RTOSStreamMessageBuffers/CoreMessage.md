# 内核到内核信息流

## 简介

消息缓冲区允许将可变长度的离散消息从中断服务程序传递到任务，或从一个任务传递到另一个任务。 例如，长度为10、20 甚至123字节的消息都可以写入和读取同一个消息缓冲区。与使用流缓冲区不同，10字节消息只能作为10字节消息读出，而不能作为单个字节读出。消息缓冲区构建在流缓冲区之上。

数据通过复制传递通过消息缓冲区 - 数据由发送方复制到缓冲区中，并通过读取从缓冲区中复制出来。另请参阅[configMESSAGE_BUFFER_LENGTH_TYPE](https://freertos.org/a00110.html#configMESSAGE_BUFFER_LENGTH_TYPE)配置参数的定义。 

与大多数FreeRTOS通信数据形式不同，流缓冲区以及消息缓冲区针对单读取单写入场景进行了优化，例如将数据从中断服务函数传递到任务，或从一个微控制器内核传递到双核CPU上的另一个内核。

通过在编译中包含*FreeRTOS/source/stream_buffer.c*源文件来启用消息缓冲区功能。 

消息缓冲区实现使用直接到任务通知。因此，使用将所调用的任务置于阻塞状态的消息缓冲区API函数可以更改调用任务的通知状态和值。

## 入门

*FreeRTOS/Demo/Common/Minimal/MessageBufferAMP.c*源文件提供了注释丰富的示例，说明如何使用消息缓冲区将可变长度数据从一个MCU内核传递到多核MCU上的另一个内核。这是一个相当高级的应用场景，但是在更简单的单核场景中创建、发送到消息缓冲区和从消息缓冲区接收的机制是相同的，与demo不同的是，不需要覆盖[sbSEND_COMPLETE()](https://freertos.org/RTOS-stream-buffer-example.html#macros)宏。

有关消息缓冲区相关 API 函数的列表，请参阅用户文档的[消息缓冲区](https://freertos.org/RTOS-message-buffer-API.html)部分，在许多情况下，包括demo正在使用的函数的代码片段。

## 调整消息缓冲区的大小 

为了使消息缓冲区能够处理可变大小的消息，每条消息的长度在消息本身之前写入消息缓冲区(这在 FreeRTOS API 函数内部发生)。长度存储在一个变量中，其类型由*FreeRTOSConfig.h*中的*configMESSAGE_BUFFER_LENGTH_TYPE*设置。 如果未定义，*configMESSAGE_BUFFER_LENGTH_TYPE*默认为size_t类型。 size_t在32位架构上通常为4字节。 因此，例如，当*configMESSAGE_BUFFER_LENGTH_TYPE*为4字节时，将10字节的消息写入消息缓冲区实际上会消耗14字节的缓冲区空间。同样，将100字节的消息写入消息缓冲区实际上将存储104字节的缓冲区空间。

## 阻塞消息的读写

[xMessageBufferReceive()](https://freertos.org/xMessageBufferReceive.html)用于从RTOS任务的消息缓冲区读取数据。[xMessageBufferReceiveFromISR()](https://freertos.org/xMessageBufferReceiveFromISR.html)用于从中断服务函数(ISR)的消息缓冲区中读取数据。 [xMessageBufferSend()](https://freertos.org/xMessageBufferSend.html)用于将数据从RTOS任务发送到消息缓冲区。[xMessageBufferSendFromISR()](https://freertos.org/xMessageBufferSendFromISR.html)用于从中断服务函数(ISR)向消息缓冲区发送数据。

如果在任务使用xMessageBufferReceive()从恰好为空的消息缓冲区读取时指定了非零阻塞时间，则该任务将被置于阻塞状态直到消息缓冲区中的数据可用，或者阻塞时间到达。

如果在任务使用xMessageBufferSend()写入恰好已满的消息缓冲区时指定了非零阻塞时间，则该任务将被置于阻塞状态直到消息缓冲区中有可用空间，或者阻塞时间到达。 

## 发送和接收完整的宏 

由于消息缓冲区建立在流缓冲区上，因此sbSEND_COMPLETE()和sbRECEIVE_COMPLETE()宏的行为与描述[流缓冲区](TaskSteam.md)上描述的完全相同。 

