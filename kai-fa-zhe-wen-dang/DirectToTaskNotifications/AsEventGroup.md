# 作为事件组

[事件组](https://freertos.org/FreeRTOS-Event-Groups.html)是一组二进制标志(位)，开发者可以对事件组进行定义。RTOS任务可以进入阻塞状态以等待组内的一个或多个标志变为活动状态。RTOS任务在处于阻塞状态时不消耗任何CPU时间。 

使用任务通知代替事件组时，使用接收任务的通知值代替事件组，接收任务通知值中的位用作事件标志，并使用[xTaskNotifyWait()](https://freertos.org/xTaskNotifyWait.html) API函数代替事件组的*xEventGroupWaitBits()* API函数。

同样，使用[xTaskNotify()](https://freertos.org/xTaskNotify.html)和[xTaskNotifyFromISR()](https://freertos.org/xTaskNotifyFromISR.html) API函数(将参数eAction设置为eSetBits)来分别代替*xEventGroupSetBits()*和*xEventGroupSetBitsFromISR()*函数来使能标志位。

与*xEventGroupSetBitsFromISR()*相比，*xTaskNotifyFromISR()*具有显着的性能优势，因为*xTaskNotifyFromISR()*完全在ISR中执行，而 *xEventGroupSetBitsFromISR()*必须将某些处理推送到RTOS守护进程任务。 

与使用事件组不同的是，接收任务不能指定期望位组合以离开阻塞状态。相反，当任何位变为活动状态时，任务将被解除阻塞，并且必须自行测试位组合。 

例程如下所示: 

```cpp
/* 本例程实现如何通过单个RTOS任务处理源自两个独立中断服务程序的事件-
一个发送中断和一个接收中断。 许多外设对两者都使用相同的处理程序。
这种情况下，外设的中断状态寄存器可以简单地与接收任务的通知值进行按位或运算。 

First bits are defined to represent each interrupt source. */
#define TX_BIT    0x01
#define RX_BIT    0x02

/* 将从中断接收通知的创建任务时定义的句柄。*/
static TaskHandle_t xHandlingTask;

/*-----------------------------------------------------------*/

/* The implementation of the transmit interrupt service routine. */
void vTxISR( void )
{
BaseType_t xHigherPriorityTaskWoken = pdFALSE;

   /* Clear the interrupt source. */
   prvClearInterrupt();

   /* Notify the task that the transmission is complete by setting the TX_BIT
   in the task's notification value. */
   xTaskNotifyFromISR( xHandlingTask,
                       TX_BIT,
                       eSetBits,
                       &xHigherPriorityTaskWoken );

   /*若xHigherPriorityTaskWoken设置为 pdTRUE，
   应执行上下文切换以确保中断直接返回到最高优先级任务。
   用于此目的的宏取决于正在使用的端口，
   可称为portEND_SWITCHING_ISR()。  */
   portYIELD_FROM_ISR( xHigherPriorityTaskWoken );
}
/*-----------------------------------------------------------*/

/* The implementation of the receive interrupt service routine is identical
except for the bit that gets set in the receiving task's notification value. */
void vRxISR( void )
{
BaseType_t xHigherPriorityTaskWoken = pdFALSE;

   /* Clear the interrupt source. */
   prvClearInterrupt();

   /* Notify the task that the reception is complete by setting the RX_BIT
   in the task's notification value. */
   xTaskNotifyFromISR( xHandlingTask,
                       RX_BIT,
                       eSetBits,
                       &xHigherPriorityTaskWoken );

   /* If xHigherPriorityTaskWoken is now set to pdTRUE then a context switch
   should be performed to ensure the interrupt returns directly to the highest
   priority task.  The macro used for this purpose is dependent on the port in
   use and may be called portEND_SWITCHING_ISR(). */
   portYIELD_FROM_ISR( xHigherPriorityTaskWoken );
}
/*-----------------------------------------------------------*/

/* The implementation of the task that is notified by the interrupt service
routines. */
static void prvHandlingTask( void *pvParameter )
{
const TickType_t xMaxBlockTime = pdMS_TO_TICKS( 500 );
BaseType_t xResult;

   for( ;; )
   {
      /* Wait to be notified of an interrupt. */
      xResult = xTaskNotifyWait( pdFALSE,    /* Don't clear bits on entry. */
                           ULONG_MAX,        /* Clear all bits on exit. */
                           &ulNotifiedValue, /* Stores the notified value. */
                           xMaxBlockTime );

      if( xResult == pdPASS )
      {
         /* A notification was received.  See which bits were set. */
         if( ( ulNotifiedValue & TX_BIT ) != 0 )
         {
            /* The TX ISR has set a bit. */
            prvProcessTx();
         }

         if( ( ulNotifiedValue & RX_BIT ) != 0 )
         {
            /* The RX ISR has set a bit. */
            prvProcessRx();
         }
      }
      else
      {
         /* Did not receive a notification within the expected time. */
         prvCheckForErrors();
      }
   }
}
```
