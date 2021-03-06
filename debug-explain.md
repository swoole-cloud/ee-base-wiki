Tracker的调试工具链目前有4种，分别是`性能分析`、`阻塞检测`、`内存泄漏`、`查看当前调用栈`。

经常有人问它们的适用场景以及区别，在此予以说明：

## 内存泄漏

是用在 Swoole、Workerman 等常驻进程模式下的工具，在 FPM 下是不存在内存泄漏一说的，它能帮助你检查这种情况：

在请求开始后向某一个全局变量（`$GLOBALS`或者类的静态属性等）追加内容，并且在请求结束的时候追加在全局变量的内容没有被`unset`掉，那么就认为有内存泄漏。

>[info] 请求开始和请求结束可以通过调用`Tracker API`来指定，具体参考「[调试器](debuger.md)」章节`手动埋点调试`。

## 阻塞检测

阻塞检测工具的准确的来说应该叫做`阻塞系统调用检测工具`，它的原理是拦截进程的所有系统调用，在开始和结束打点取时间差，如果大于配置的时间（默认`10ms`）就把当前的 PHP 调用堆栈上报到 Tracker 的后台，可以方便的定位到调用栈阻塞在哪个系统调用，耗时多少毫秒，通过具体的系统调用我们能判断出进程是锁等待、还是磁盘 IO 问题、网络 IO 问题等。

但是这个工具无法检测出卡死的问题，具体可以参考微课程的这篇文章[教你秒级定位PHP卡死问题](https://course.swoole-cloud.com/article/2)

## 查看当前调用栈

对于阻塞检测工具无法发现的卡死问题，需要用`查看当前调用栈`工具来定位到底卡死在哪个 PHP 函数了，调用堆栈是什么。

## 性能分析

是最直观的一种性能分析方式，因为他可以生成一整张的调用图，他的原理是拦截所有的 PHP 函数调用，并且在开始和结束打点取时间差。

它是非常好的`定位hot函数`的工具，方便的定位到最耗时的函数是哪个，但是无法看到调用栈，也不知道具体的系统调用，每次调用的耗时也不知道。

## 到底该用哪个工具

一般性能调优的话用`性能分析`工具就够用，生产环境突然产生了性能问题最先用性能分析查看，先大概定位问题，如果请求阻塞非常久（也就是卡死）可以直接用`查看当前调用栈`工具查看，如果请求只是变慢了就用`阻塞检测工具`。

当然也可以挨个使用一遍，多维度发现问题。