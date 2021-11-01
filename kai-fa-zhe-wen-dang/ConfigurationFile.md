# FreeRTOSConfig.h

FreeRTOS使用FreeRTOSConfig.h对应用工程进行配置。每个FreeRTOS应用程序在其预处理器包含路径中都必须有一个FreeRTOSConfig.h头文件。FreeRTOSConfig.h 根据所创建的应用程序定制RTOS内核。因此，该配置文件特定于应用程序，而不是RTOS，并且应该位于应用程序目录中，而不是仅仅位于RTOS内核源代码目录中。

RTOS源代码包含的每个demo应用程序都有相应的FreeRTOSConfig.h文件。 一些demo很旧，不包含所有可用的配置选项。省略的配置选项设置为RTOS源文件中的默认值。

下面是一个典型的FreeRTOSConfig.h定义:

```cpp
#ifndef FREERTOS_CONFIG_H
#define FREERTOS_CONFIG_H

/* Here is a good place to include header files that are required across
your application. */
#include "something.h"

#define configUSE_PREEMPTION                    1
#define configUSE_PORT_OPTIMISED_TASK_SELECTION 0
#define configUSE_TICKLESS_IDLE                 0
#define configCPU_CLOCK_HZ                      60000000
#define configSYSTICK_CLOCK_HZ                  1000000
#define configTICK_RATE_HZ                      250
#define configMAX_PRIORITIES                    5
#define configMINIMAL_STACK_SIZE                128
#define configMAX_TASK_NAME_LEN                 16
#define configUSE_16_BIT_TICKS                  0
#define configIDLE_SHOULD_YIELD                 1
#define configUSE_TASK_NOTIFICATIONS            1
#define configTASK_NOTIFICATION_ARRAY_ENTRIES   3
#define configUSE_MUTEXES                       0
#define configUSE_RECURSIVE_MUTEXES             0
#define configUSE_COUNTING_SEMAPHORES           0
#define configUSE_ALTERNATIVE_API               0 /* Deprecated! */
#define configQUEUE_REGISTRY_SIZE               10
#define configUSE_QUEUE_SETS                    0
#define configUSE_TIME_SLICING                  0
#define configUSE_NEWLIB_REENTRANT              0
#define configENABLE_BACKWARD_COMPATIBILITY     0
#define configNUM_THREAD_LOCAL_STORAGE_POINTERS 5
#define configSTACK_DEPTH_TYPE                  uint16_t
#define configMESSAGE_BUFFER_LENGTH_TYPE        size_t

/* Memory allocation related definitions. */
#define configSUPPORT_STATIC_ALLOCATION             1
#define configSUPPORT_DYNAMIC_ALLOCATION            1
#define configTOTAL_HEAP_SIZE                       10240
#define configAPPLICATION_ALLOCATED_HEAP            1
#define configSTACK_ALLOCATION_FROM_SEPARATE_HEAP   1

/* Hook function related definitions. */
#define configUSE_IDLE_HOOK                     0
#define configUSE_TICK_HOOK                     0
#define configCHECK_FOR_STACK_OVERFLOW          0
#define configUSE_MALLOC_FAILED_HOOK            0
#define configUSE_DAEMON_TASK_STARTUP_HOOK      0

/* Run time and task stats gathering related definitions. */
#define configGENERATE_RUN_TIME_STATS           0
#define configUSE_TRACE_FACILITY                0
#define configUSE_STATS_FORMATTING_FUNCTIONS    0

/* Co-routine related definitions. */
#define configUSE_CO_ROUTINES                   0
#define configMAX_CO_ROUTINE_PRIORITIES         1  //协程的优先级为0~(configMAX_CO_ROUTINE_PRIORITIES-1)

/* Software timer related definitions. */
#define configUSE_TIMERS                        1
#define configTIMER_TASK_PRIORITY               3
#define configTIMER_QUEUE_LENGTH                10
#define configTIMER_TASK_STACK_DEPTH            configMINIMAL_STACK_SIZE

/* Interrupt nesting behaviour configuration. */
#define configKERNEL_INTERRUPT_PRIORITY         [dependent of processor]
#define configMAX_SYSCALL_INTERRUPT_PRIORITY    [dependent on processor and application]
#define configMAX_API_CALL_INTERRUPT_PRIORITY   [dependent on processor and application]

/* Define to trap errors during development. */
#define configASSERT( ( x ) ) if( ( x ) == 0 ) vAssertCalled( __FILE__, __LINE__ )

/* FreeRTOS MPU specific definitions. */
#define configINCLUDE_APPLICATION_DEFINED_PRIVILEGED_FUNCTIONS 0
#define configTOTAL_MPU_REGIONS                                8 /* Default value. */
#define configTEX_S_C_B_FLASH                                  0x07UL /* Default value. */
#define configTEX_S_C_B_SRAM                                   0x07UL /* Default value. */
#define configENFORCE_SYSTEM_CALLS_FROM_KERNEL_ONLY            1

/* ARMv8-M secure side port related definitions. */
#define secureconfigMAX_SECURE_CONTEXTS         5

/* Optional functions - most linkers will remove unused functions anyway. */
#define INCLUDE_vTaskPrioritySet                1
#define INCLUDE_uxTaskPriorityGet               1
#define INCLUDE_vTaskDelete                     1
#define INCLUDE_vTaskSuspend                    1
#define INCLUDE_xResumeFromISR                  1
#define INCLUDE_vTaskDelayUntil                 1
#define INCLUDE_vTaskDelay                      1
#define INCLUDE_xTaskGetSchedulerState          1
#define INCLUDE_xTaskGetCurrentTaskHandle       1
#define INCLUDE_uxTaskGetStackHighWaterMark     0
#define INCLUDE_xTaskGetIdleTaskHandle          0
#define INCLUDE_eTaskGetState                   0
#define INCLUDE_xEventGroupSetBitFromISR        1
#define INCLUDE_xTimerPendFunctionCall          0
#define INCLUDE_xTaskAbortDelay                 0
#define INCLUDE_xTaskGetHandle                  0
#define INCLUDE_xTaskResumeFromISR              1

/* A header file that defines trace macro can be included here. */

#endif /* FREERTOS_CONFIG_H */
```

## 'config'参数

### configUSE_PREEMPTION

设置为1以使能抢占式RTOS调度器，或设置为0使用合作式RTOS调度器。

### configUSE_PORT_OPTIMISED_TASK_SELECTION

某些FreeRTOS port有两种选择要执行的下一个任务的方案-通用方案和特定于该port的方案。

通用方案: 

*   当configUSE_PORT_OPTIMISED_TASK_SELECTION设置为0或未实现特定于port的方案时使用;
*   在所有FreeRTOS port中通用;
*   完全用C语言编写，使其执行效率低于特定于port的方案;
*   不对可用优先级的最大数量施加限制;

特定于port方案:

*   并非对所有方案都通用;
*   当configUSE_PORT_OPTIMISED_TASK_SELECTION设置为1时被采用;
*   依赖于一个或多个特定于体系结构的汇编指令(通常是计数前导零[CLZ]或等效指令，因此只能与适配的体系结构一起使用;
*   比通用方案更高效;
*   通常将可用优先级的最大数量限制为32。

### configUSE_TICKLESS_IDLE

将configUSE_TICKLESS_IDLE设置为1以使用[低功耗无滴答模式](https://freertos.org/low-power-tickless-rtos.html)，或设置为0以保持滴答中断始终运行。并非所有FreeRTOS port都提供低功耗无滴答实现。

### configUSE_IDLE_HOOK

如果您希望使用空闲任务，则设置为1，否则设置为0。

### configUSE_MALLOC_FAILED_HOOK

每次创建任务、队列或信号量时，内核都会调用pvPortMalloc()从堆中分配内存。为此，FreeRTOS源码提供四个示例内存分配方案。这些方案分别在heap_1.c、heap_2.c、heap_3.c、heap_4.c和heap_5.c源文件中实现。configUSE_MALLOC_FAILED_HOOK仅在使用这三个示例方案之一时才相关。

malloc()失败的hook函数是一个hook(或回调)函数，如果定义和配置该函数，将在pvPortMalloc()返回NULL时被调用。仅当FreeRTOS堆内存不足以使请求的分配成功时，才会返回NULL。

如果configUSE_MALLOC_FAILED_HOOK设置为1，则应用程序必须定义malloc() failed hook函数。如果configUSE_MALLOC_FAILED_HOOK设置为0，则malloc() failed hook函数即便被定义了也不会被调用。Malloc() failed hook函数必须具有如下所示的声明。

```cpp
void vApplicationMallocFailedHook( void );
```

### configUSE_DAEMON_TASK_STARTUP_HOOK

如果configUSE_TIMERS和configUSE_DAEMON_TASK_STARTUP_HOOK都设置为1，那么应用程序必须定义一个hook函数，这种函数具有如下所示的确切声明。当RTOS守护进程任务(也称为[定时服务任务](https://freertos.org/RTOS-software-timer-service-daemon-task.html))第一次执行时，hook函数将被调用一次。任何需要运行RTOS的应用程序初始化代码都可以放在hook函数中。

```cpp
void void vApplicationDaemonTaskStartupHook( void );
```

### configUSE_TICK_HOOK

若使用[滴答hook](https://freertos.org/a00016.html#TickHook)，该宏设置为1，否则设置为0。

### configCPU_CLOCK_HZ

输入用于生成滴答中断的外设的内部时钟的频率(以Hz为单位) - 通常与驱动内部CPU的时钟相同。该值是配置定时器外设所必要的。

### configSYSTICK_CLOCK_HZ

仅适用于ARM Cortex-M port的可选参数。

默认情况下，ARM Cortex-M port从Cortex-M SysTick定时器生成RTOS滴答中断。大多数Cortex-M MCU以与自身相同的频率运行SysTick计时器 - 在这种情况下，不需要configSYSTICK_CLOCK_HZ并且应该保持未定义。若SysTick定时器以与MCU内核不同的频率计时，则将configCPU_CLOCK_HZ设置为MCU时钟频率，并将 configSYSTICK_CLOCK_HZ设置为SysTick时钟频率。

### configTICK_RATE_HZ

表示RTOS滴答中断的频率。

滴答中断用于测量时间。因此，更高的滴答频率意味着可以以更高的精度测量时间。然而，高滴答频率也意味着RTOS内核将占用更多的CPU处理时间，因此程序执行效率较低。RTOS demo应用程序滴答频率都为1000Hz。该频率用于测试RTOS内核，高于正常值。

多个任务可以共享相同的优先级。RTOS调度程序将通过每个滴答周期的任务切换来在相同优先级的任务之间共享处理器时间。因此，高滴答频率也将具有减少给予每个任务的"时间片"的效果。

### configMAX_PRIORITIES

表示应用程序任务可用的优先级数。任意数量的任务可以共享相同的优先级。协程单独确定优先级 - 具体参阅configMAX_CO_ROUTINE_PRIORITIES。

每个可用的优先级在RTOS内核中占用少量RAM，因此不应将此值设置为高于应用程序实际需要的值。

如果configUSE_PORT_OPTIMISED_TASK_SELECTION设置为1，则最大允许值将受到限制。

### configMINIMAL_STACK_SIZE

表示空闲任务使用的堆栈大小。通常，这不应从使用的port的demo应用程序提供的FreeRTOSConfig.h文件中设置的值中减少。

与xTaskCreate()和xTaskCreateStatic()函数的堆栈大小参数一样，堆栈大小以字而不是字节指定。如果每个对象单位是32位堆栈，那么100个堆栈大小意味着400字节。

### configMAX_TASK_NAME_LEN

创建任务时赋予任务的描述性名称的最大允许长度。该长度包括NULL终止字节在内。

### configUSE_TRACE_FACILITY

若希望包含额外的结构成员和函数以帮助执行可视化和跟踪，该宏设置为1。

### configUSE_STATS_FORMATTING_FUNCTIONS

将configUSE_TRACE_FACILITY和configUSE_STATS_FORMATTING_FUNCTIONS设置为1，从而创建包含vTaskList()和vTaskGetRunTimeStats()函数。否则，不能创建包含vTaskList()和vTaskGetRunTimeStates()的函数。

### configUSE_16_BIT_TICKS

时间以"滴答"来衡量--表示自RTOS内核启动以来滴答中断执行的次数。滴答计数保存在TickType_t类型的变量中。

将configUSE_16_BIT_TICKS定义为1会导致TickType_t被定义为16位无符号类型。将configUSE_16_BIT_TICKS定义为0会导致TickType_t被定义为32位无符号类型。

使用16位类型将大大提高8位和16位体系架构的芯片的性能，但最大可时间段只能限制为65535个"滴答"。因此，假设滴答频率为250Hz，使用16位定时器任务可以延迟或阻塞的最长时间为262秒，而使用32位计数器时为17179869秒。

### configIDLE_SHOULD_YIELD

该参数能够控制处于空闲优先级的任务的行为。仅在以下情况下有效：

1.   正在使用抢占式调度器；
2.   应用程序创建以空闲优先级运行的任务。

如果configUSE_TIME_SLICING设置为1（或未定义），则共享相同优先级的任务将进行时间切片。如果没有任何任务被抢占，那么可能会假设给定优先级的每个任务将被分配等量的处理时间 - 如果优先级高于空闲优先级，则情况确实如此。

当任务共享空闲优先级时，行为可能略有不同。如果configIDLE_SHOULD_YIELD设置为1，那么如果空闲优先级的任何其他任务准备运行，空闲任务将立即让步。该机制确保了当应用程序任务可用于调度时，在空闲任务上花费的时间最少。 但是，可能会产生不良影响（取决于您的应用程序的需要），如下所示：

![配置文件](https://freertos.org/fr-content-src/uploads/2018/07/idleyield.gif "共享空闲优先级特殊情况")

上图显示了四个任务都以空闲优先级运行的执行模式。 任务A、B 和 C 是应用程序任务。 任务I是空闲任务。上下文切换在时间T0、T1、...、T6 定期发生。当空闲任务产生时，任务A开始执行--但空闲任务已经消耗了当前的一些时间片。 这导致任务I和任务A有效地共享相同的时间片。因此，应用程序任务B和C比应用程序任务A获得更多的处理时间。

这种情况可以通过以下方法避免：

*   如果合适，使用空闲hook代替空闲优先级的单独任务。
*   以高于空闲优先级的优先级创建应用程序任务。
*   将 configIDLE_SHOULD_YIELD 设置为 0。

将configIDLE_SHOULD_YIELD设置为0可防止空闲任务在其时间片结束之前放弃处理时间。这确保了空闲优先级的所有任务都被分配了等量的处理时间（如果没有任务被抢占）-- 但代价是分配给空闲任务的总处理时间的比例更大。

### configUSE_TASK_NOTIFICATIONS

将configUSE_TASK_NOTIFICATIONS设置为1(或保持configUSE_TASK_NOTIFICATIONS未定义)将在编译中包含直接到任务的通知功能及其关联的API。

将configUSE_TASK_NOTIFICATIONS设置为0将从编译中排除直接到任务通知功能及其关联的API。

当编译中包含直接到任务的通知时，每个任务都会额外消耗8个字节的RAM。

### configTASK_NOTIFICATION_ARRAY_ENTRIES

每个RTOS任务都有一组任务通知。configTASK_NOTIFICATION_ARRAY_ENTRIES设置数组中的索引数。

在FreeRTOS V10.4.0之前的版本，任务只有一个通知值，而不是一组值。因此，为了向后兼容，configTASK_NOTIFICATION_ARRAY_ENTRIES如果未定义，则默认为 1。

### configUSE_MUTEXES

设置为1以在编译中包含互斥功能，或设置为0以在编译中禁用互斥功能。开发者应熟悉与FreeRTOS功能相关的互斥锁和二进制信号量之间的差异。

### configUSE_RECURSIVE_MUTEXES

设置为1以在编译中包含递归互斥锁功能，或设置为0以从编译中禁用递归互斥锁功能。

### configUSE_COUNTING_SEMAPHORES

设置为1以在编译中包括计数信号量功能，或设置为0以在编译中禁用计数信号量功能。

### configUSE_ALTERNATIVE_API

设置为1以在编译中包含"替代"队列函数，或设置为0以在编译中禁用"替代"队列函数。queue.h头文件中包含了替代 API。替代 API已弃用，不应在新的应用程序中使用。

### configCHECK_FOR_STACK_OVERFLOW

[堆栈溢出检测](https://freertos.org/Stacks-and-stack-overflow-checking.html)描述了该参数的使用。

### configQUEUE_REGISTRY_SIZE

队列注册表有两个功能，这两个功能都与RTOS内核感知调试相关联：

1.   允许将文本名称与队列相关联，以便在调试GUI中轻松识别队列。
2.   包含调试器定位每个注册队列和信号量所需的信息。

除非使用RTOS内核感知调试器，否则队列注册表没有任何用途。

configQUEUE_REGISTRY_SIZE定义了可以注册的队列和信号量的最大数量。只需要注册所需的RTOS内核感知调试器查看的相关队列和信号量。有关更多信息，请参阅 [vQueueAddToRegistry()](https://freertos.org/vQueueAddToRegistry.html)和[vQueueUnregisterQueue()](https://freertos.org/vQueueUnregisterQueue.html)的API参考文档。

### configUSE_QUEUE_SETS

设置为1以使能队列集功能（在多个队列和信号量上阻塞或挂起的能力），或设置为0以阻塞队列集功能。

### configUSE_TIME_SLICING

默认情况下(如果configUSE_TIME_SLICING未定义，或者configUSE_TIME_SLICING定义为1)，FreeRTOS使用优先级抢占式调度和时间轮询。这意味着RTOS调度程序将始终运行处于就绪状态的最高优先级任务，并会在每个RTOS滴答中断时在同等优先级的任务之间切换。如果configUSE_TIME_SLICING设置为0，那么RTOS调度程序仍将运行处于就绪状态的最高优先级任务，但不会仅仅因为发生滴答中断而在同等优先级的任务之间切换。

### configUSE_NEWLIB_REENTRANT

如果configUSE_NEWLIB_REENTRANT设置为1，那么应用程序将为每个创建的任务分配一个newlib reent结构。

注意Newlib支持已成为主流应用程序需求，但FreeRTOS 维护者并未使用。FreeRTOS不对由此产生的newlib操作负责。开发者必须熟悉newlib并且必须提供必要的能够回退的版本。

### configENABLE_BACKWARD_COMPATIBILITY

FreeRTOS.h头文件包含一组#define宏，这些宏将8.0.0版之前的FreeRTOS版本中使用的数据类型名称映射到FreeRTOS 8.0.0版中使用的名称。宏允许应用程序代码将使用的FreeRTOS版本从8.0.0之前的版本更新到8.0.0之后的版本，而无需修改。在FreeRTOSConfig.h中将configENABLE_BACKWARD_COMPATIBILITY设置为0会从编译中排除宏，这样可以验证没有使用8.0.0之前的RTOS版本。

### configNUM_THREAD_LOCAL_STORAGE_POINTERS

设置每个任务的线程本地存储阵列中的索引数。

### configSTACK_DEPTH_TYPE

设置用于在调用xTaskCreate()时指定堆栈深度的类型，以及使用堆栈大小的各种其他地方(例如，当返回堆栈高水位线时)。

旧版本的FreeRTOS使用UBaseType_t类型的变量指定堆栈大小，但发现这在8位微控制器上难以实现。configSTACK_DEPTH_TYPE使应用程序开发人员能够指定要使用的类型来消除该限制。

### configMESSAGE_BUFFER_LENGTH_TYPE

FreeRTOS消息缓冲区使用configMESSAGE_BUFFER_LENGTH_TYPE类型的变量来存储每条消息的长度。如果configMESSAGE_BUFFER_LENGTH_TYPE未定义，则默认为 size_t。如果存储在消息缓冲区中的消息永远不超过255字节，那么将configMESSAGE_BUFFER_LENGTH_TYPE定义为uint8_t，能够在32位微控制器上为每条消息节省3个字节。同样，如果消息缓冲区中存储的消息永远不会大于65535字节，那么将configMESSAGE_BUFFER_LENGTH_TYPE定义为uint16_t，能够在32位微控制器上为每个消息节省2个字节。

### configSUPPORT_STATIC_ALLOCATION

如果configSUPPORT_STATIC_ALLOCATION设置为1，则可以使用开发者指定的RAM创建RTOS对象。

如果configSUPPORT_STATIC_ALLOCATION设置为0，则只能使用从FreeRTOS堆分配的RAM创建RTOS对象

如果configSUPPORT_STATIC_ALLOCATION未定义，则默认为0。

如果configSUPPORT_STATIC_ALLOCATION设置为1，则开发者还必须提供两个回调函数：vApplicationGetIdleTaskMemory()提供给RTOS空闲任务使用的内存，vApplicationGetTimerTaskMemory()(如果configUSE_TIMERS设置为1)提供给RTOS使用的内存RTOS守护进程/定时器服务任务，示例如下:

```cpp
/* configSUPPORT_STATIC_ALLOCATION is set to 1, so the application must provide an
implementation of vApplicationGetIdleTaskMemory() to provide the memory that is
used by the Idle task. */
void vApplicationGetIdleTaskMemory( StaticTask_t **ppxIdleTaskTCBBuffer,
                                    StackType_t **ppxIdleTaskStackBuffer,
                                    uint32_t *pulIdleTaskStackSize )
{
/* If the buffers to be provided to the Idle task are declared inside this
function then they must be declared static - otherwise they will be allocated on
the stack and so not exists after this function exits. */
static StaticTask_t xIdleTaskTCB;
static StackType_t uxIdleTaskStack[ configMINIMAL_STACK_SIZE ];

    /* Pass out a pointer to the StaticTask_t structure in which the Idle task's
    state will be stored. */
    *ppxIdleTaskTCBBuffer = &xIdleTaskTCB;

    /* Pass out the array that will be used as the Idle task's stack. */
    *ppxIdleTaskStackBuffer = uxIdleTaskStack;

    /* Pass out the size of the array pointed to by *ppxIdleTaskStackBuffer.
    Note that, as the array is necessarily of type StackType_t,
    configMINIMAL_STACK_SIZE is specified in words, not bytes. */
    *pulIdleTaskStackSize = configMINIMAL_STACK_SIZE;
}
/*-----------------------------------------------------------*/

/* configSUPPORT_STATIC_ALLOCATION and configUSE_TIMERS are both set to 1, so the
application must provide an implementation of vApplicationGetTimerTaskMemory()
to provide the memory that is used by the Timer service task. */
void vApplicationGetTimerTaskMemory( StaticTask_t **ppxTimerTaskTCBBuffer,
                                     StackType_t **ppxTimerTaskStackBuffer,
                                     uint32_t *pulTimerTaskStackSize )
{
/* If the buffers to be provided to the Timer task are declared inside this
function then they must be declared static - otherwise they will be allocated on
the stack and so not exists after this function exits. */
static StaticTask_t xTimerTaskTCB;
static StackType_t uxTimerTaskStack[ configTIMER_TASK_STACK_DEPTH ];

    /* Pass out a pointer to the StaticTask_t structure in which the Timer
    task's state will be stored. */
    *ppxTimerTaskTCBBuffer = &xTimerTaskTCB;

    /* Pass out the array that will be used as the Timer task's stack. */
    *ppxTimerTaskStackBuffer = uxTimerTaskStack;

    /* Pass out the size of the array pointed to by *ppxTimerTaskStackBuffer.
    Note that, as the array is necessarily of type StackType_t,
    configTIMER_TASK_STACK_DEPTH is specified in words, not bytes. */
    *pulTimerTaskStackSize = configTIMER_TASK_STACK_DEPTH;
}

Examples of the callback functions that must be provided by the application to

supply the RAM used by the Idle and Timer Service tasks if configSUPPORT_STATIC_ALLOCATION

is set to 1.
```

### configSUPPORT_DYNAMIC_ALLOCATION

如果configSUPPORT_DYNAMIC_ALLOCATION设置为1，则可以使用从FreeRTOS堆自动分配的RAM创建RTOS对象。

如果configSUPPORT_DYNAMIC_ALLOCATION未定义，它将默认为1。

### configTOTAL_HEAP_SIZE

FreeRTOS堆中可用的RAM总量。

仅当configSUPPORT_DYNAMIC_ALLOCATION设置为1且应用程序使用FreeRTOS源码提供的示例内存分配方案之一时，才会使用此值。有关更多详细信息，请参阅内存配置部分。

### configAPPLICATION_ALLOCATED_HEAP

默认情况下，[FreeRTOS堆](https://freertos.org/a00111.html)由FreeRTOS声明并由链接器放置在内存中。将configAPPLICATION_ALLOCATED_HEAP设置为1时，允许应用程序开发者声明堆，并将堆放在指定的内存位置。

若使用heap_1.c、heap_2.c或heap_4.c，且configAPPLICATION_ALLOCATED_HEAP设置为1，则应用程序开发者必须提供具有确切名称和维度的uint8_t数组，如下所示。该数组将用作FreeRTOS堆。数组如何放置在特定的内存位置取决于所使用的编译器。

```cpp
uint8_t ucHeap[ configTOTAL_HEAP_SIZE ];
```

### configSTACK_ALLOCATION_FROM_SEPARATE_HEAP

如果configSTACK_ALLOCATION_FROM_SEPARATE_HEAP设置为1，则使用xTaskCreate或xTaskCreateRestricted API创建的任何任务的堆栈使用pvPortMallocStack分配并使用vPortFreeStack函数释放。开发者需要提供pvPortMallocStack和vPortFreeStack函数的线程安全实现。这使程序能够从单独的内存区域(可能是不同于FreeRTOS堆的另一个堆)为任务分配堆栈。

如果configSTACK_ALLOCATION_FROM_SEPARATE_HEAP未定义，则默认为 0。

pvPortMallocStack和vPortFreeStack函数的示例实现如下：

```cpp
void * pvPortMallocStack( size_t xWantedSize  )
{
    /* Allocate a memory block of size xWantedSize. The function used for
     * allocating memory must be thread safe. */
    return MyThreadSafeMalloc( xWantedSize );
}

void vPortFreeStack( void * pv )
{
    /* Free the memory previously allocated using pvPortMallocStack. The
     * function used for freeing the memory must be thread safe. */
    MyThreadSafeFree( pv );
}
```

### configGENERATE_RUN_TIME_STATS

[Run Time Stats](https://freertos.org/rtos-run-time-stats.html)说明了此参数的使用。

### configUSE_CO_ROUTINES

设置为1以在编译中包含协程相关功能的函数，设置为0从编译中禁用协程。如需使用协程，croutine.c必须包含在项目中。

### configMAX_CO_ROUTINE_PRIORITIES

表示应用程序协程可用的优先级数。任意数量的协程可以共享相同的优先级。任务被单独划分优先级 - 请参阅configMAX_PRIORITIES。

### configUSE_TIMERS

设置为1以使能软件定时器功能，或设置为0以禁用软件定时器功能。

### configTIMER_TASK_PRIORITY

用来设置软件定时器服务/守护程序任务的优先级。

### configTIMER_QUEUE_LENGTH

用来设置软件定时器命令队列的长度。

### configTIMER_TASK_STACK_DEPTH

用来设置分配给软件计时器服务/守护程序任务的堆栈深度。

### configKERNEL_INTERRUPT_PRIORITY configMAX_SYSCALL_INTERRUPT_PRIORITY and configMAX_API_CALL_INTERRUPT_PRIORITY

包含configKERNEL_INTERRUPT_PRIORITY设置的port包括ARM Cortex-M3、PIC24、dsPIC、PIC32、SuperH和RX600。包含configMAX_SYSCALL_INTERRUPT_PRIORITY 设置的port包括PIC32、RX600、ARM Cortex-A和ARM Cortex-M port。

ARM Cortex-M3和ARM Cortex-M4开发者请注意本节末尾的特别说明。

configMAX_API_CALL_INTERRUPT_PRIORITY是configMAX_SYSCALL_INTERRUPT_PRIORITY的新名称，仅供较新的port使用。

configKERNEL_INTERRUPT_PRIORITY应设置为最低优先级。

请注意，在以下讨论中，只能从中断服务例程中调用以"FromISR"结尾的API函数。

$\underline{对于仅实现configKERNEL_INTERRUPT_PRIORITY的port}$

configKERNEL_INTERRUPT_PRIORITY设置RTOS内核本身使用的中断优先级。调用API函数的中断也必须以此优先级执行。不调用API函数的中断可以以更高的优先级执行，因此它们的执行永远不会被RTOS内核活动延迟（在硬件本身的限制内）。

$\underline{对于同时支持configKERNEL_INTERRUPT_PRIORITY和configMAX_SYSCALL_INTERRUPT_PRIORITY的port:}$

configKERNEL_INTERRUPT_PRIORITY设置RTOS内核本身使用的中断优先级。

configMAX_SYSCALL_INTERRUPT_PRIORITY设置可以调用中断安全FreeRTOS API函数的最高中断优先级。

完整的中断嵌套模型是通过将configMAX_SYSCALL_INTERRUPT_PRIORITY设置为高于configKERNEL_INTERRUPT_PRIORITY来实现的。 这意味着FreeRTOS内核不会完全禁用中断，即使在关键部分也是如此。 此外，这是在没有分段内核体系结构缺点的情况下实现的。但是注意，某些微控制器架构将（在硬件中）在接受新中断时禁用中断 - 这意味着在硬件接受中断和FreeRTOS代码重新启用中断之间的短时间内，不可避免地会禁用中断。

不调用API函数的中断可以以高于configMAX_SYSCALL_INTERRUPT_PRIORITY的优先级执行，因此永远不会被RTOS内核执行延迟。

例如，假设一个具有8个中断优先级的微控制器 - 0是最低的，7是最高的（请参阅本节末尾的ARM Cortex-M3用户特别说明）。下图描述了如果将两个配置常量设置为4和0，则在每个优先级可以做什么和不能做什么，如下所示：

![配置文件](https://freertos.org/fr-content-src/uploads/2018/07/Interrupt-priorities-interrupt-nesting.jpg "中断优先级配置示例")

这些配置参数的配置可以活用中断处理：

*   可以根据系统中的任何其他任务编写中断处理"任务"并确定其优先级。这些是由中断唤醒的任务。中断服务程序(ISR)本身应尽可能短--它只是获取数据然后唤醒高优先级处理程序任务。ISR然后直接返回到被唤醒的处理程序任务中 - 因此中断处理在时间上是连续的，就像它在ISR本身中完成一样。 这样做的好处是在处理程序任务执行时所有中断都保持启用状态。
*   实现configMAX_SYSCALL_INTERRUPT_PRIORITY的port更进一步 - 允许完全嵌套的模型。其中RTOS内核中断优先级和configMAX_SYSCALL_INTERRUPT_PRIORITY 之间的中断可以嵌套并进行适用的API调用。RTOS内核活动永远不会延迟优先级高于configMAX_SYSCALL_INTERRUPT_PRIORITY的中断。
*   运行在最大系统调用优先级之上的ISR永远不会被RTOS内核本身屏蔽，因此其响应能力不受RTOS内核功能的影响。该机制非常适用于需要非常高时间精度的中断 - 例如执行电机换向的中断。但是，此类ISR无法使用FreeRTOS API函数。

要使用此方案，应用程序设计必须遵守以下规则：必须将使用FreeRTOS API的任何中断设置为与RTOS内核相同的优先级（由configKERNEL_INTERRUPT_PRIORITY宏配置），或等于或低于具备此功能的port的configMAX_SYSCALL_INTERRUPT_PRIORITY。

ARM Cortex-M3和ARM Cortex-M4用户的特别说明：请参考[ARM Cortex-M设备上的中断优先级设置](https://freertos.org/RTOS-Cortex-M3-M4.html)。请记住 ARM Cortex-M3内核使用数字低优先级数字来表示高优先级中断，这看起来违反直觉并且很容易忘记！如果您希望为中断分配低优先级，请勿为其分配优先级0（或其他低数值），因为这可能导致中断实际上在系统中具有最高优先级 - 因此，如果应用程序优先级高于configMAX_SYSCALL_INTERRUPT_PRIORITY，可能会使您的系统崩溃。

ARM Cortex-M3内核的最低优先级实际上是255 - 但是不同的ARM Cortex-M3供应商实现了不同数量的优先级位并提供了期望以不同方式指定优先级的库函数。例如，在STM32上，可以在ST驱动程序库调用中指定的最低优先级实际上是15 - 而理论上可以指定的最高优先级是0。

### configASSERT

configASSERT()宏的语义与标准C assert()宏相同。如果传入configASSERT()的参数为零，则触发assertion。

在FreeRTOS源文件中调用configASSERT()以检查应用程序如何使用FreeRTOS。建议在FreeRTOS应用程序开发中定义configASSERT()。

示例定义调用vAssertCalled()，传入触发configASSERT()调用的文件名和行号(__FILE__ 和 __LINE__ 是大多数编译器提供的标准宏)。上述仅用于演示，因为 vAssertCalled()不是FreeRTOS库函数，实际应用中可定义configASSERT()。

以防止应用程序进一步执行的方式定义configASSERT()。原因有两点: 1.允许设置应用程序的调试断点；2.超过可执行区间的断点可能会导致崩溃。

```cpp
/* Define configASSERT() to call vAssertCalled() if the assertion fails.  The assertion
has failed if the value of the parameter passed into configASSERT() equals zero. */
#define configASSERT ( x )     if( ( x ) == 0 ) vAssertCalled( __FILE__, __LINE__ )
```

如果在调试器的控制下运行FreeRTOS，则configASSERT()可定义为仅禁用中断并处于循环中，如下所示。这将导致在运行所在程序行上的代码调试失败 - 暂停调试器并立刻定位到有问题的行，以便分析代码运行失败的原因。

```cpp
/* Define configASSERT() to disable interrupts and sit in a loop. */
#define configASSERT ( x )     if( ( x ) == 0 ) { taskDISABLE_INTERRUPTS(); for( ;; ); }
```

### configINCLUDE_APPLICATION_DEFINED_PRIVILEGED_FUNCTIONS

configINCLUDE_APPLICATION_DEFINED_PRIVILEGED_FUNCTIONS仅用于FreeRTOS的MPU。

如果configINCLUDE_APPLICATION_DEFINED_PRIVILEGED_FUNCTIONS设置为1，那么应用程序开发者须提供一个名为"application_defined_privileged_functions.h"的头文件。该头文件可以实现应用程序开发者需要在特权模式下执行的功能。请注意，尽管具有.h扩展名，但头文件应包含C函数的实现，而不仅仅是函数的声明。

在"application_defined_privileged_functions.h"中实现的函数必须分别使用prvRaisePrivilege()函数和portRESET_PRIVILEGE()宏来保存和恢复处理器的特权状态。例如，如果库提供的print函数访问不受应用程序控制的RAM，因此无法分配给受内存保护的用户模式任务，则可以使用以下代码将打印函数封装在特权函数中: 

```cpp
void MPU_debug_printf( const char *pcMessage )
{
/* State the privilege level of the processor when the function was called. */
BaseType_t xRunningPrivileged = prvRaisePrivilege();

    /* Call the library function, which now has access to all RAM. */
    debug_printf( pcMessage );

    /* Reset the processor privilege level to its original value. */
    portRESET_PRIVILEGE( xRunningPrivileged );
}
```

该方案应该只在开发期间使用，而不是在发布后使用，因为绕过了内存保护。

### configTOTAL_MPU_REGIONS

ARM Cortex-M4微控制器的FreeRTOS MPU（内存保护单元）端口支持具有16个MPU区域的设备。对于具有16个MPU区域的设备，将configTOTAL_MPU_REGIONS设置为16。如果未定义，则默认为8。

### configTEX_S_C_B_FLASH

TEX、可共享(S)、可缓存(C) 和可缓冲(B)位定义内存类型，并在必要时定义MPU区域的可缓存和可共享属性。configTEX_S_C_B_FLASH允许应用程序覆盖闪存的MPU 区域的TEX、可共享(S)、可缓存(C) 和可缓冲(B)位的默认值。若未定义，则默认为0x07UL，这意味着TEX=000, S=1, C=1, B=1。

### configTEX_S_C_B_SRAM

TEX、可共享(S)、可缓存(C)和可缓冲(B)位定义内存类型，并在必要时定义MPU区域的可缓存和可共享属性。configTEX_S_C_B_SRAM允许应用程序覆盖RAM的MPU区域的TEX、可共享(S)、可缓存(C)和可缓冲(B)位的默认值。 如果未定义，则默认为0x07UL，这意味着TEX=000, S=1, C=1, B=1。

### configENFORCE_SYSTEM_CALLS_FROM_KERNEL_ONLY

configENFORCE_SYSTEM_CALLS_FROM_KERNEL_ONLY可定义为1，以防止任何源自内核代码外部的优先级提升（除了在进入中断时由硬件本身执行的优先级提升）。当 FreeRTOSConfig.h中的configENFORCE_SYSTEM_CALLS_FROM_KERNEL_ONLY设置为1时，需要从链接描述文件中导出变量 __syscalls_flash_start__ 和 __syscalls_flash_end__ 以分别指示系统调用内存的起始地址和结束地址。建议将其定义为1，以使应用程序获得最大的安全性。

### secureconfigMAX_SECURE_CONTEXTS

该宏仅用于ARMv8-M安全侧port。

从ARMv8-M MCU (ARM Cortex-M23、Cortex-M33 和 Cortex-M55)的非安全端调用安全函数的任务有两个上下文--一个在非安全端，一个在安全端。在FreeRTOS v10.4.5之前的架构版本，ARMv8-M安全端port分配在运行时引用安全端上下文的结构体。从FreeRTOS V10.4.5开始，这些结构体在编译时静态分配。 secureconfigMAX_SECURE_CONTEXTS设置静态分配的安全上下文的数量。如果未定义，secureconfigMAX_SECURE_CONTEXTS默认为8。仅在ARMv8-M微控制器的非安全端使用FreeRTOS代码的应用程序，例如在安全端运行第三方代码的应用程序，不需要此常量。

## INCLUDE参数

以"INCLUDE"开头的宏允许应用程序未使用的实时内核组件在编译中排除。这可确保RTOS不会使用任何超出特定嵌入式应用程序所需的ROM或RAM。

每个宏都采用以下形式:

```cpp
INCLUDE_FunctionName
```

其中FunctionName表示需要选择性地排除的API函数(或函数集)。如虚包含API函数，将宏设置为1，若排除该函数，将宏设置为0。例如，要包含vTaskDelete() API 函数，使用：

```cpp
#define INCLUDE_vTaskDelete    1
```

排除vTaskDelete() API 函数，使用：

```cpp
#define INCLUDE_vTaskDelete    0
```
