# 任务相关API函数

## 任务基本API

1.[xTaskCreate()](https://freertos.org/a00125.html): 

函数声明: 

```cpp
 BaseType_t xTaskCreate(    TaskFunction_t pvTaskCode,
                            const char * const pcName,
                            configSTACK_DEPTH_TYPE usStackDepth,
                            void *pvParameters,
                            UBaseType_t uxPriority,
                            TaskHandle_t *pxCreatedTask
                          );
```

函数功能: **创建一个新任务并将其添加到准备运行的任务列表中**。*configSUPPORT_DYNAMIC_ALLOCATION*必须在*FreeRTOSConfig.h*中设置为1，或者未定义（在这种情况下，它将默认为 1），以便此RTOS API函数可用。

每个任务都需要分配RAM, 该RAM用于**保存任务状态并用作任务的堆栈**。如果任务是使用*xTaskCreate()*创建的，那么所需的RAM会从FreeRTOS堆中自动分配。 如果使用xTaskCreateStatic()创建任务，则RAM由开发者指定，因此可以在编译时静态分配。 有关详细信息，请参阅静态与动态分配页面。

函数参数: 

*   *pvTaskCode*: 指向任务入口函数的指针(实现任务的函数名)。

    任务通常以死循环的方式实现。实现任务的函数绝不能尝试返回或退出，但任务可以删除自身。

*   *pcName*: 任务的描述性名称。主要用于方便调试，也可用于获取任务句柄。

    任务名称的最大长度由FreeRTOSConfig.h中的configMAX_TASK_NAME_LEN定义。

*   *usStackDepth*: 分配用作任务堆栈的字数(根据堆栈位宽，16位或者32位)。例如，如果堆栈为16位宽且usStackDepth为100，则将分配200个字节用作任务的堆栈。再举一个例子，如果堆栈是32位宽并且usStackDepth是400，那么将分配1600个字节用作任务的堆栈。

    堆栈深度乘以堆栈宽度不得超过可包含在size_t类型变量中的最大值。

*   *pvParameters*: 作为参数传递给创建的任务的值。

    如果pvParameters设置为变量的地址，那么在创建的任务执行时该变量必须仍然存在-因此传递堆栈变量的地址是无效的。

*   *uxPriority*: 创建的任务将执行的优先级。

    支持MPU的系统可以通过在uxPrriority中设置位portPRIVILEGE_BIT来选择在特权（系统）模式下创建任务。例如，要创建优先级为2的特权任务，请将 uxPriority设置为( 2 | portPRIVILEGE_BIT )。
