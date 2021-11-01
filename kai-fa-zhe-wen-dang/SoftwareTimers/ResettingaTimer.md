# 定时器重置
可以将已经开始运行的定时器进行重置。重置定时器会导致定时器重新计算timeout，因此timeout与重置时定时器的时间有关，而不是相对于定时器最初启动的时间。下图中描述了该过程，其中定时器1是一个单次计时器，其周期等于5秒。 

在该示例中，假设应用程序在按下某个键时打开LCD背光，并且背光保持亮起直到5秒后没有按下任何键。定时器1用于在5秒过后关闭LCD背光。 

![软件定时器](https://freertos.org/fr-content-src/uploads/2018/07/resetting-a-FreeRTOS-software-timer.png "定时器重置的表现")