---
layout: post
title: Node event loop 扒皮
date:   2017-06-19 21:28:32 +0800
tags: event loop    
---

时间循环的流程如下
```
┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
└───────────────────────┘

```
每个phase都维持着一个FIFO的回调函数队列。

### Phases
- `timers`: this phase executes callbacks scheduled by setTimeout() and setInterval().
- `I/O callbacks`: executes almost all callbacks with the exception of close callbacks, the ones scheduled by timers, and setImmediate().
- `idle`, `prepare`: only used internally.
- `poll`: retrieve new I/O events; node will block here when appropriate.
- `check`: setImmediate() callbacks are invoked here.
- `close callbacks`: e.g. `socket.on('close', ...)`.


#### timer
Technically, the `poll` phase controls when timers are executed.


#### I/0 callbacks

执行一些系统操作的回调函数，如TCP错误类型。

eg：当尝试连接时，如果一个TCP套接字收到了`ECONNREFUSED`， 一些linux系统可能希望等待错误报告。这将会在本阶段内入队等待执行。


#### poll

本阶段有两个主要功能：
1. 执行队列里已到期的timer，然后
2. 处理poll队列里的事件

如果没有timer调度，将会发生以下两种之一：
- 如果poll队列不为空，则继续同步迭代器队列内的回调函数直到exhausted或是达到系统硬性限制。
- 如果poll队列为空， 将会发生以下两种之一。
    * 如果脚本被`setImmediate()`调度，时间循环将会结束`poll`阶段并进入`check`阶段执行被调度的脚本。
    * 如果脚本不是被`setImmediate()`调度，事件循环将会等待回调函数加入队列，然后即时之行之。

一旦poll队列为空，时间循环会检查到达threshold的timers。如果一个或多个timer ready了，时间循环将会包装会timer阶段执行timer的回调函数。

#### check

本阶段允许在poll阶段结束后立即执行callbacks。如果poll阶段转为idle并且被setImmediate()入队，时间循环将直接进入`check`阶段而不是等待`poll`事件。

`setImmediate()`实际上是在事件循环中一个单独的阶段运行的一个特殊的timer。使用一个`libuv`的api来调度在poll阶段完成之后的callbacks的执行

通常来说，在代码执行过程中，事件循环将最终hit到poll阶段（poll阶段也会等待for incoming connection, request，etc）。但如果一个callback被`setImmediate()`调度并且poll阶段变为idle，it（poll）将直接结束并进入check阶段而非等待poll事件。


#### close callbacks

如果一个socket或是handle被突然关闭(e.g. `socket.destroy()`),`close`事件将会被emit到本阶段。其他情况将会被emit到`process.nextTick()`。


----
####  `setImmediate()`  vs `setTimeout()`
这两者很相似，但是行为表现不同 depend on 何时调用。

- `setImmediate()`被设计为在当前poll阶段完成时执行
- `setTimeout()`调度一段脚本在达到minimum的threshold时执行


`setImmediate()`over`setTimeout()`的最主要优点在于：
`setImmediate()`的执行顺序比任何在`I/O`阶段调度的timer高。



----

## `process.nextTick()`
任何时候你都可以在一个给定的阶段里调用`process.nextTick()`,所有传入`process.nextTick()`的callbacks都会在事件循环continue前resolved.这可以创建一些很不好的解决方案，因为你可以利用递归`process.nextTick()`手动starve将你的`I/O`,这种做法将防止事件循环进入poll阶段。




####  `process.nextTick()` vs `setImmediate()`  

- `process.nextTick()`在本阶段立即解除
- `setImmediate()`事件循环中的接下来的iteration或是`tick`中解除





----

## Why use `process.nextTick()`
有两个主要原因：
1. 允许使用者处理错误，清初任何不必要的资源，或是可能在下次事件循环时尝试再次请求
2. 在下次事件循环前，有时候有必要在调用栈释放后，允许一个callback执行









ps:
1. 如果可能的话，调用setTimeout时，尽量使用相同的超时值
2. 尽量用process.nextTick 来代替 setTimeout(fn ,0)
