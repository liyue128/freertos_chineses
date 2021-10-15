# RTOS任务通知

## 简述

每个RTOS任务都有一组任务通知。每个任务通知都有一个"挂起"或"未挂起"的值为32位的通知状态。常量[configTASK_NOTIFICATION_ARRAY_ENTRIES](https://freertos.org/a00110.html#configTASK_NOTIFICATION_ARRAY_ENTRIES)设置任务通知数组中的索引数。在FreeRTOS V10.4.0之前，任务只有一个而不是一系列通知。

直接到任务的通知表示直接发送到任务的事件，而不是通过中间对象(如队列、事件组或信号量)间接发送到任务的事件。向任务发送直接到任务通知会将目标任务通知的状态设置为"挂起"。 正如任务可以在中间对象(例如信号量)上阻塞以等待该信号量可用一样，任务可以在任务通知上阻塞以等待该通知的状态变为挂起。

向任务发送直接到任务的通知也可以选择以下方式之一更新目标通知的值: 

*   无论接收任务是否读取了被覆盖的值，都会覆盖该值;
*   在接收任务已读取被覆盖的值的前提下，覆盖该值;
*   在值中将1位或者多位置1;
*   将值递增(加1);

调用[xTaskNotifyWait()/xTaskNotifyWaitIndexed()](https://freertos.org/xTaskNotifyWait.html)读取通知值会将该通知的状态清除为"未挂起"。通知状态也可以调用[xTaskNotifyStateClear()/xTaskNotifyStateClearIndexed()](https://freertos.org/xTaskNotifyStateClear.html)被显式地设置为"未挂起"。

注意: 数组中的每个通知都是独立运行的--一个任务一次只能阻塞数组中的一个通知，并且不会被发送到任何其他数组索引的通知从而避免错误地解除阻塞。

RTOS任务通知功能默认情况下启用，可以在FreeRTOSConfig.h中将configUSE_TASK_NOTIFICATIONS设置为0从而在编译过程中忽略(每个任务每个数组索引节省8个字节)。

**$\color{red}{重要说明}$**: FreeRTOS [Stream和Message Buffers](https://freertos.org/RTOS-stream-message-buffers.html)在数组索引为0处使用任务通知。如果要在调用Stream或Message Buffer API函数时保持任务通知的状态，则在数组索引处使用任务通知大于0.

## 性能优势与使用限制

任务通知的灵活性允许在需要创建单独队列、二进制信号量、计数信号量或事件组的情况下使用。与使用中间对象(例如二进制信号量)解除任务阻塞相比，使用直接通知解除阻塞RTOS任务的速度要快45%，并且使用的RAM更少。但是，这些性能优势需要一些使用场景限制: 

1.   RTOS任务通知只能在仅将一个任务作为事件的接收者时使用。实际应用的大多数用例都满足这种条件，例如中断解除阻塞任务将处理通过中断接收数据。
2.   仅在使用RTOS任务通知代替队列的情况下: 虽然接收任务可以在阻塞状态等待通知(因此不消耗任何 CPU 时间)，但发送任务不能在阻塞状态等待一个完成无法立刻实现的发送。

## 使用用例

通知使用xTaskNotifyIndexed()和xTaskNotifyGiveIndexed() API函数(及其中断安全等效函数)发送，并保持挂起状态。直到接收RTOS任务调用 xTaskNotifyWaitIndexed()或ulTaskNotifyTakeIndexed() API函数。这些API函数中的每一个都有一个没有"索引"前缀的等效函数。 非"索引"版本始终对数组索引 0处的任务通知进行操作。例如xTaskNotifyGive(TargetTask)等效于xTaskNotifyGiveIndexed(TargetTask, 0) - 两者都在处理TargetTask的任务句柄的任务的索引 0 处增加任务通知。

### 例程

[Using RTOS task notifications as a light weight binary semaphore](https://freertos.org/RTOS_Task_Notification_As_Binary_Semaphore.html)

[Using RTOS task notifications as a light weight counting semaphore](https://freertos.org/RTOS_Task_Notification_As_Counting_Semaphore.html)

[Using RTOS task notifications as a light weight event group](https://freertos.org/RTOS_Task_Notification_As_Event_Group.html)

[Using RTOS task notifications as a light weight mailbox](https://freertos.org/RTOS_Task_Notification_As_Mailbox.html)
