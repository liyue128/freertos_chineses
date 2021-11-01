# 源码结构

每个RTOS端口都带有一个预配置的demo应用程序，该应用程序的编译不仅包含必要的RTOS源文件，还包括必要的RTOS头文件。建议将所提供的demo用作所有基于 FreeRTOS的新应用程序的基础。本部分能够帮助定位和理解所提供的项目源码。

## 基本目录结构

FreeRTOS包括每个处理器端口和每个demo应用程序的源代码。将所有端口放在同一个文件夹中极大地简化了分发，但文件的数量非常大。但是目录结构非常简单，FreeRTOS实时内核仅包含在3个文件中(如果需要软件定时器、事件组或协同例程功能，则需要其他文件)。

从根目录开始，分为两个子文件夹: FreeRTOS和FreeRTOS-Plus。 这些如下所示: 

```
+-FreeRTOS-Plus    Contains FreeRTOS+ components and demo projects.
|
+-FreeRTOS         Contains the FreeRTOS real time kernel source
                   files and demo projects
```

FreeRTOS-Plus目录树包含多个描述其内容的README文件。

## FreeRTOS 内核目录结构

FreeRTOS内核主要源文件和demo项目包含在两个子目录中，如下所示: 

```
FreeRTOS
    |
    +-Demo      Contains the demo application projects.
    |
    +-Source    Contains the real time kernel source code.
```

**RTOS核心代码包含在三个文件中: tasks.c、queue.c和list.c**。这三个文件位于FreeRTOS/Source目录中。同一目录包含两个名为timers.c和croutine.c的可选文件，分别实现**软件定时器**和**协程**功能。

每个受支持的处理器架构都需要少量特定于架构的RTOS代码，称为RTOS可移植层，位于FreeRTOS/Source/Portable/[compiler]/[architecture]子目录中，其中 [compiler]和[architecture]分别表示编译器和芯片架构。

由于[内存管理页面](https://freertos.org/a00111.html)所述的原因，示例堆分配方案也位于可移植层。 各种示例 heap_x.c 文件位于 FreeRTOS/Source/portable/MemMang 目录中。

可移植层目录示例: 

*   如果将TriCore 1782架构与GCC编译器一起使用;

    指定TriCore的文件(port.c)位于FreeRTOS/Source/Portable/GCC/TriCore_1782目录中。可以忽略或删除除FreeRTOS/Source/Portable/MemMang之外的所有其他 FreeRTOS/Source/Portable子目录。

*   如果使用带有IAR编译器的Renesas RX600架构;

    指定RX600的文件(port.c)位于FreeRTOS/Source/Portable/IAR/RX600目录中。可以忽略或删除除FreeRTOS/Source/Portable/MemMang之外的所有其他FreeRTOS/Source/Portable子目录。

*   其他port同理；

FreeRTOS/Source目录的结构如下所示。

```
FreeRTOS
    |
    +-Source        The core FreeRTOS kernel files
        |
        +-include   The core FreeRTOS kernel header files
        |
        +-Portable  Processor specific code.
            |
            +-Compiler x    All the ports supported for compiler x
            +-Compiler y    All the ports supported for compiler y
            +-MemMang       The sample heap implementations
```

FreeRTOS源码还包含每个处理器架构和编译器的demo应用程序。大多数demo应用程序代码对所有port都是通用的，并包含在FreeRTOS/Demo/Common/Minimal目录中(位于FreeRTOS/Demo/Common/Full目录中的代码是遗留代码，仅供PC端口使用).

其余的FreeRTOS/Demo子目录包含用于构建单个demo应用程序的预配置项目。命名目录以指示相关的架构和编译器。

demo目录示例:

*   如果构建面向英飞凌TriBoard硬件的TriCore GCC demo应用程序: 

    TriCore 演示应用项目文件位于FreeRTOS/Demo/TriCore_TC1782_TriBoard_GCC目录中。FreeRTOS/Demo目录中包含的所有其他子目录(**Common目录除外**)都可以忽略或删除。

*   如果构建针对RX62N RDK硬件的Renesas RX6000 IAR demo应用程序:

    IAR工作区文件位于FreeRTOS/Demo/RX600_RX62N-RDK_IAR目录中。FreeRTOS/Demo目录中包含的所有其他子目录(Common 目录除外)都可以忽略或删除。

*   其他port同理；

FreeRTOS/Demo目录的结构如下所示。

```
FreeRTOS
    |
    +-Demo
        |
        +-Common    The demo application files that are used by all the demos.
        +-Dir x     The demo application build files for port x
        +-Dir y     The demo application build files for port y
```

## 构建自己的应用程序

浏览demo目录以确保项目配置的示例已经存在，其中包含正确的RTOS内核源文件和正确的编译器选项集，以最少的用户工作量进行构建。因此，建议通过修改现有的预配置demo应用程序来创建新应用程序。首先采用匹配的demo应用程序以确保可以实现最简洁的构建来完成，然后用自己的应用程序源文件增量替换FreeRTOS/Demo 目录中项目中包含的文件。
