# 任务流缓存

## 简介

流缓冲区允许将字节流从中断服务函数传递到任务，或从一个任务传递到另一个任务。字节流可以是任意长度，不一定有开始或结束。可以一次性写入任意数量的字节，也可以一次性读取任意数量的字节。数据可通过复制进行传递-数据由发送方复制到缓冲区中，并通过复制从缓冲区中读取出来。 

与大多数FreeRTOS通信数据形式不同，流缓冲区针对单读取方单写入方场景进行了优化，例如将数据从中断服务函数传递到任务，或从在核CPU中从一个微控制器内核传递到另一个内核。 

通过在编译代码中加入*FreeRTOS/source/stream_buffer.c*源文件启用流缓冲区功能。

流缓冲区实现使用[直接到任务的通知](https://freertos.org/RTOS-task-notifications.html)。 因此，使用将调用任务置于阻塞状态的流缓冲区的API函数可以更改调用任务的通知状态和值。

## 入门

*FreeRTOS/Demo/Common/Minimal/StreamBufferInterrupt.c*源文件提供了如何使用流缓冲区将数据从中断服务函数传递到任务的大量示例。
有关流缓冲区相关API函数的列表，请参阅用户文档的[流缓冲区](https://freertos.org/RTOS-stream-buffer-API.html)部分。 

## 阻塞读取和触发级别 

[xStreamBufferReceive()](https://freertos.org/xStreamBufferReceive.html)用于从RTOS任务的流缓冲区中读取数据。[xStreamBufferReceiveFromISR()](https://freertos.org/xStreamBufferReceiveFromISR.html)用于从中断服务函数(ISR)的流缓冲区中读取数据。

*xStreamBufferReceive()*允许指定阻塞时间。如果在任务使用*xStreamBufferReceive()*从恰好为空的流缓冲区读取时指定了非零阻塞时间，则该任务将被置于阻塞状态(不消耗任何CPU时间)，直到流缓冲区中有指定数量的数据可用，或者阻塞时间到达。在等待数据的任务从阻塞状态移除之前必须位于流缓冲区中的数据大小称为流缓冲区的触发级别。例如：

*   若任务在读取触发级别为1的空流缓冲区时被阻塞，则当单个字节写入缓冲区或任务的阻塞时间到达时，将解除任务阻塞。
*   若任务在读取触发级别为10的空流缓冲区时被阻塞，则在流缓冲区包含至少10个字节或任务的阻塞时间到达时，任务阻塞解除。 

如果在达到触发级别之前读取任务的阻塞时间到达，则不论实际上流缓存区有多少字节，该任务仍将读取流缓存区。

**$\underline{注意}$**

*   将触发级别设置为0是无效的。若将触发级别设置为0，则触发级别将会被设置为1。
*   大于流缓冲区大小的触发级别也是无效的。 

流缓冲区的触发级别是在创建流缓冲区时初始化的，可以使用[xStreamBufferSetTriggerLevel()](https://freertos.org/xStreamBufferSetTriggerLevel.html) API函数对触发级别进行更改。

## 阻塞写入

[xStreamBufferSend()](https://freertos.org/xStreamBufferSend.html)能将数据从RTOS任务发送到流缓冲区。[xStreamBufferSendFromISR()](https://freertos.org/xStreamBufferSendFromISR.html)可以从中断服务函数(ISR)向流缓冲区发送数据。

若在任务使用xStreamBufferSend()向已满的流缓冲区执行写入时指定非零阻塞时间，则该任务将被置于阻塞状态，直到流缓冲区中有可用空间，或者阻塞时间到达。

## 发送和接收完整的宏 

参考[blog on using message buffers for dual core - core to core communication](https://freertos.org/2020/02/simple-multicore-core-to-core-communication-using-freertos-message-buffers.html).

### sbSEND_COMPLETED()(或sbSEND_COMPLETED_FROM_ISR())

sbSEND_COMPLETED()是一个在数据写入流缓冲区时调用的宏。该宏接受一个参数，即更新的流缓冲区的句柄。

sbSEND_COMPLETED()验证是否由于流缓存区等待数据写入而导致任务阻塞。如果是这样的话，将任务的阻塞状态解除。

应用开发者能够根据在*FreeRTOSConfig.h*中定义sbSEND_COMPLETED()代替默认宏定义。这在使用流缓存区在多核处理器中用作各个内核之间的数据传递是非常有用的。在这种情况下，sbSEND_COMPLETED()可用来在其他CPU内核中生成中断，中断的服务函数可以使用*xStreamBufferSendCompletedFromISR()* API函数检查并在必要时解除阻塞等待数据的任务。*FreeRTOS/Demo/Common/Minimal/MessageBufferAMP.c*源文件提供了该情况下的大量例程。 

### sbRECEIVE_COMPLETED()(或sbRECEIVE_COMPLETED_FROM_ISR())

sbRECEIVE_COMPLETED()是sbSEND_COMPLETED()用作接收的等效项。该宏会被用来读取流缓冲区数据。该宏检查是否有任务阻塞在流缓冲区上以等待缓冲区出现可用空间，若存在有效空间，则将任务解除阻塞状态。 与sbSEND_COMPLETED()一样，通过在 FreeRTOSConfig.h中定义函数替代默认函数。
