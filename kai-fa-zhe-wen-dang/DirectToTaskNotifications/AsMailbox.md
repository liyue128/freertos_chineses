# 用作邮箱

RTOS任务通知可用于向任务发送数据，但其实现比RTOS队列要严格得多，因为: 

1.  只能发送32位值;
2.  该值保存为接收任务的通知值，且任一时刻只能有一个通知值。

因此，短语"轻量级邮箱"优先于"轻量级队列"使用。 任务的通知值是邮箱值。

使用*xTaskNotify()*(或*xTaskNotifyIndexed()*)和*xTaskNotifyFromISR()*(或*xTaskNotifyIndexedFromISR()*) API函数将数据发送到任务，其中参数eAction设置为 eSetValueWithOverwrite或eSetValueWithoutOverwrite。如果eAction设置为eSetValueWithOverwrite，则即使接收任务已经有一个未决通知，接收任务的通知值也会更新。如果eAction设置为eSetValueWithoutOverwrite，则接收任务的通知值仅在接收任务尚未有未决通知时才更新-因为更新通知值将在接收任务处理之前覆盖先前的值。 

任务可以使用*xTaskNotifyWait()*(或*xTaskNotifyWaitIndexed()*)读取自己的通知值。 
