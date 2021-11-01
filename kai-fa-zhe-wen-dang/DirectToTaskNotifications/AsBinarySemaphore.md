# 用作二进制信号量

使用直接通知解除RTOS任务的阻塞比使用二进制信号量解除阻塞任务快45%且使用更少的RAM。本节描述具体实现方式。

二进制信号量是最大计数为1的信号量，因此加上"二进制"的前缀。如果信号量可用，任务只能"获取"信号量，且信号量仅在其计数为1时可用。

当使用任务通知代替二进制信号量时，用接收任务的[通知值](https://freertos.org/RTOS-task-notifications.html)代替二进制信号量的计数值，并用[ulTaskNotifyTake()](https://freertos.org/ulTaskNotifyTake.html)(或*ulTaskNotifyTakeIndexed()*) API函数代替信号量的*xSemaphoreTake()* API函数。*ulTaskNotifyTake()*函数的参数xClearOnExit设置为pdTRUE，因此每次接收通知时计数值都会返回零-模拟二进制信号量。

同样，使用[xTaskNotifyGive()](https://freertos.org/xTaskNotifyGive.html)(或xTaskNotifyGiveIndexed())或[vTaskNotifyGiveFromISR()](https://freertos.org/vTaskNotifyGiveFromISR.html)(或*vTaskNotifyGiveIndexedFromISR()*)函数代替信号量的 *xSemaphoreGive()*和*xSemaphoreGiveFromISR()*函数。

例程如下所示: 

```cpp
/* This is an example of a transmit function in a generic
peripheral driver.  An RTOS task calls the transmit function,
then waits in the Blocked state (so not using an CPU time)
until it is notified that the transmission is complete.  The
transmission is performed by a DMA, and the DMA end interrupt
is used to notify the task. */

/* Stores the handle of the task that will be notified when the
transmission is complete. */
static TaskHandle_t xTaskToNotify = NULL;

/* The index within the target task's array of task notifications
to use. */
const UBaseType_t xArrayIndex = 1;

/* The peripheral driver's transmit function. */
void StartTransmission( uint8_t *pcData, size_t xDataLength )
{
    /* At this point xTaskToNotify should be NULL as no transmission
    is in progress.  A mutex can be used to guard access to the
    peripheral if necessary. */
    configASSERT( xTaskToNotify == NULL );

    /* Store the handle of the calling task. */
    xTaskToNotify = xTaskGetCurrentTaskHandle();

    /* Start the transmission - an interrupt is generated when the
    transmission is complete. */
    vStartTransmit( pcData, xDatalength );
}
/*-----------------------------------------------------------*/

/* The transmit end interrupt. */
void vTransmitEndISR( void )
{
BaseType_t xHigherPriorityTaskWoken = pdFALSE;

    /* At this point xTaskToNotify should not be NULL as
    a transmission was in progress. */
    configASSERT( xTaskToNotify != NULL );

    /* Notify the task that the transmission is complete. */
    vTaskNotifyGiveIndexedFromISR( xTaskToNotify,
                                   xArrayIndex,
                                   &xHigherPriorityTaskWoken );

    /* There are no transmissions in progress, so no tasks
    to notify. */
    xTaskToNotify = NULL;

    /* If xHigherPriorityTaskWoken is now set to pdTRUE then a
    context switch should be performed to ensure the interrupt
    returns directly to the highest priority task.  The macro used
    for this purpose is dependent on the port in use and may be
    called portEND_SWITCHING_ISR(). */
    portYIELD_FROM_ISR( xHigherPriorityTaskWoken );
}
/*-----------------------------------------------------------*/

/* The task that initiates the transmission, then enters the
Blocked state (so not consuming any CPU time) to wait for it
to complete. */
void vAFunctionCalledFromATask( uint8_t ucDataToTransmit,
                                size_t xDataLength )
{
uint32_t ulNotificationValue;
const TickType_t xMaxBlockTime = pdMS_TO_TICKS( 200 );

    /* Start the transmission by calling the function shown above. */
    StartTransmission( ucDataToTransmit, xDataLength );

    /* Wait to be notified that the transmission is complete.  Note
    the first parameter is pdTRUE, which has the effect of clearing
    the task's notification value back to 0, making the notification
    value act like a binary (rather than a counting) semaphore.  */
    ulNotificationValue = ulTaskNotifyTakeIndexed( xArrayIndex,
                                                   pdTRUE,
                                                   xMaxBlockTime );

    if( ulNotificationValue == 1 )
    {
        /* The transmission ended as expected. */
    }
    else
    {
        /* The call to ulTaskNotifyTake() timed out. */
    }
}
```
