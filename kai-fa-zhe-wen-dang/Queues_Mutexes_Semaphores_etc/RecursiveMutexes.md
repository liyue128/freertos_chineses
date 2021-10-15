# FreeRTOS Recursive Mutexes

详细内容参考[Blocking on Multiple RTOS Objects](https://freertos.org/Pend-on-multiple-rtos-objects.html)

## FreddRTOS递归互斥

递归使用的互斥锁可以被持有者重复"使用"。在所有者为每个xSemaphoreTakeRecursive()请求成功调用xSemaphoreGiveRecursive()之前，互斥量不会被再次使用。例如，如果一个任务成功地"接受"同一个互斥锁5次，那么该互斥锁将无法用于其他任务，直到准确地"返还"了五次互斥锁。

这种类型的信号量使用优先级继承机制，因此一旦不再需要该信号量，"接受"信号量的任务必须始终"返还"信号量。

互斥类型信号量不能在中断服务例程中使用。

互斥锁不能在中断里面使用，因为: 

*   互斥锁包括一个优先级继承机制，只有在互斥量是从任务中给出和获取时才有意义，而不是中断。

*   中断不能通过阻塞使互斥锁保护的资源变为可用。
