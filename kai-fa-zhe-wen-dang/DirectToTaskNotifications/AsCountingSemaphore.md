# 用作计数信号量

计数信号量是一种值信号量，其计数值最小为零，最大值为创建信号量时设置的值。如果信号量可用，任务只能"take"信号量，且信号量仅在其计数大于零时才可用。

使用任务通知代替计数信号量时，使用接收任务的[通知值](https://freertos.org/RTOS-task-notifications.html)代替计数信号量的计数值，并使用[ulTaskNotifyTake()](https://freertos.org/ulTaskNotifyTake.html)（或ulTaskNotifyTakeIndexed()）API函数代替信号量的xSemaphoreTake( ) API函数。ulTaskNotifyTake()函数的参数xClearOnExit设置为pdFALSE，因此每次接收通知时计数值仅递减(而不是清除)-模拟计数信号量。

同样，使用[xTaskNotifyGive()](https://freertos.org/xTaskNotifyGive.html)（或 xTaskNotifyGiveIndexed()）和[vTaskNotifyGiveFromISR()](https://freertos.org/vTaskNotifyGiveFromISR.html)（或vTaskNotifyGiveIndexedFromISR()）函数代替信号量的 xSemaphoreGive()和xSemaphoreGiveFromISR()函数.

下面的第一个示例使用接收任务的通知值作为计数信号量。第二个示例提供了更实用、更高效的实现。 

**Example 1:**

```cpp
/* 不直接处理中断的中断处理程序，而是将处理推迟到高优先级的RTOS任务。
   ISR使用RTOS任务通知来解锁RTOS任务并增加RTOS任务的通知值。  */
void vANInterruptHandler( void )
{
BaseType_t xHigherPriorityTaskWoken;

    /* Clear the interrupt. */
    prvClearInterruptSource();

    /* xHigherPriorityTaskWoken必须初始化为 pdFALSE。
    若调用 vTaskNotifyGiveFromISR()解除对处理任务的阻塞，
    且处理任务的优先级高于当前运行任务的优先级，
    则xHigherPriorityTaskWoken将自动设置为pdTRUE。  */
    xHigherPriorityTaskWoken = pdFALSE;

    /* 解除对处理任务的阻塞，以便任务可以执行中断所需的任何处理。 
    xHandlingTask是在任务创建时生成的任务句柄。 
    vTaskNotifyGiveFromISR()还会增加接收任务的通知值。 */
    vTaskNotifyGiveFromISR( xHandlingTask, &xHigherPriorityTaskWoken );

    /* 若xHigherPriorityTaskWoken此时设置为pdTRUE，则强制进行上下文切换。 
    用于执行此操作的宏取决于端口，可以称为portEND_SWITCHING_ISR。  */
    portYIELD_FROM_ISR( xHigherPriorityTaskWoken );
}
/*-----------------------------------------------------------*/

/* A task that blocks waiting to be notified that the peripheral
needs servicing. */
void vHandlingTask( void *pvParameters )
{
BaseType_t xEvent;
const TickType_t xBlockTime = pdMS_TO_TICS( 500 );
uint32_t ulNotifiedValue;

    for( ;; )
    {
        /* 通过阻塞以等待通知。这里RTOS任务通知被用作计数信号量。
        任务的通知值在ISR每次调用vTaskNotifyGiveFromISR()时递增，
        在RTOS任务每次调用ulTaskNotifyTake()时递减-
        因此实际上保留了未完成中断数的计数。
        第一个参数设置为pdFALSE，所以通知值只递减，不清零，
        一次处理一个延迟中断事件。关于更实用的方法，参考示例 2。 */
        ulNotifiedValue = ulTaskNotifyTake( pdFALSE,
                                            xBlockTime );

        if( ulNotifiedValue > 0 )
        {
            /* Perform any processing necessitated by the interrupt. */
            xEvent = xQueryPeripheral();

            if( xEvent != NO_MORE_EVENTS )
            {
                vProcessPeripheralEvent( xEvent );
            }
        }
        else
        {
            /* Did not receive a notification within the expected
            time. */
            vCheckForErrorConditions();
        }
    }
}
```

**Example 1:**

本例程实现了一个更实用、更高效的RTOS任务。在此实现中，从ulTaskNotifyTake()返回的值用于了解必须处理多少未完成的ISR事件，从而允许每次调用 ulTaskNotifyTake()时将RTOS任务的通知计数清零。假设中断服务程序(ISR)如示例1所示。 

```cpp
/* 需要使用的目标任务的任务通知数组中的索引。 */
const UBaseType_t xArrayIndex = 0;

/* A task that blocks waiting to be notified that the peripheral
needs servicing. */
void vHandlingTask( void *pvParameters )
{
BaseType_t xEvent;
const TickType_t xBlockTime = pdMS_TO_TICS( 500 );
uint32_t ulNotifiedValue;

    for( ;; )
    {
        /* 和例程1一样，阻塞等待来自 ISR 的通知。 
        然而，首个参数设置为pdTRUE，将任务的通知值清除为 0，
        这意味着必须在再次调用ulTaskNotifyTake()之前需处理
        每个未完成的延迟中断事件。 */
        ulNotifiedValue = ulTaskNotifyTakeIndexed( xArrayIndex,
                                                   pdTRUE,
                                                   xBlockTime );

        if( ulNotifiedValue == 0 )
        {
            /* Did not receive a notification within the expected
            time. */
            vCheckForErrorConditions();
        }
        else
        {
            /* ulNotifiedValue holds a count of the number of
            outstanding interrupts.  Process each in turn. */
            while( ulNotifiedValue > 0 )
            {
                xEvent = xQueryPeripheral();

                if( xEvent != NO_MORE_EVENTS )
                {
                    vProcessPeripheralEvent( xEvent );
                    ulNotifiedValue--;
                }
                else
                {
                    break;
                }
            }
        }
    }
}
```
