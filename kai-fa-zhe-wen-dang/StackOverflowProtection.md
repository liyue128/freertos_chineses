# 栈溢出保护

## 栈的用法

每个任务都被分配相应的堆栈。如果使用[xTaskCreate()](https://freertos.org/a00125.html)创建任务，则用作任务堆栈的内存会从FreeRTOS堆中自动分配，并通过传递给xTaskCreate() API函数的参数确定大小。如果使用[xTaskCreateStatic()](https://freertos.org/xTaskCreateStatic.html)创建任务，则用作任务堆栈的内存由应用程序开发者预先分配。堆栈溢出通常是由于应用程序不稳定而造成的。FreeRTOS提供了两种可选机制，可用于协助检测和纠正此类错误。通过配置configCHECK_FOR_STACK_OVERFLOW宏选择相应的机制。

注意这些选项仅适用于不分段内存映射的架构。此外，某些处理器可能会在RTOS内核溢出检查发生之前生成错误或异常以响应堆栈崩溃。如果 configCHECK_FOR_STACK_OVERFLOW未设置为0，则应用程序必须提供堆栈溢出hook函数。hook函数必须调用vApplicationStackOverflowHook()，并使用以下声明：

```cpp
void vApplicationStackOverflowHook( TaskHandle_t xTask,
                                    signed char *pcTaskName );
```

xTask和pcTaskName参数分别将不规范任务的句柄和名称传递给钩子函数。但是请注意，根据溢出的严重程度，这些参数本身可能会损坏，在这种情况下，可以直接检查pxCurrentTCB变量。

堆栈溢出检查引入了上下文切换开销，因此仅建议在开发或测试阶段使用。

## 堆栈溢出检测 - 方法1

在RTOS内核将任务交换出运行状态后，该任务堆栈可能会达到其最大（最深）值，因为此时堆栈将包含任务上下文。RTOS内核可以检查处理器堆栈指针是否保留在有效堆栈空间内。 如果堆栈指针包含超出有效堆栈范围的值，则调用堆栈溢出挂钩函数。

这种方法的运行速度很快快，但不能保证捕获所有的堆栈溢出情况。 将configCHECK_FOR_STACK_OVERFLOW设置为1以使用此方法。

## 堆栈溢出检测 - 方法2

当一个任务第一次被创建时，它的堆栈被一个已知值填充。 当将任务从运行状态退出时，RTOS 内核可以检查有效堆栈范围内的最后16个字节，以确保这些已知值没有被任务或中断活动覆盖。如果这16个字节中的任何一个与初始值不等，就会调用堆栈溢出钩子函数。
