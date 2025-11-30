---
title: Event Loop
date: 2023-10-03
tags:
 - 前端
 - JavaScript
categories: 前端
keywords: 
 - Event Loop
 - 事件循环
---

## 背景
**Event Loop**，意如其名：*事件循环 / 事件轮询*

- *JS*是单线程执行
	- 不能同时执行*JS*和*DOM*渲染
- 所以就有了==异步==这个解决方案
- 异步要基于回调来实现
- **Event Loop** 就是这个异步回调实现方案

---

## 浏览器环境
### Event Loop 的过程

- 同步代码，一行一行的在*Call stack*中执行
- 遇到异步代码，先”记录“起来，等待（耗时时间）在 *Web APIs* / *Node APIs*中执行。
- *Web APIs* 执行完成后，异步任务的回调函数就可以放入到*Callback Queue*中
- 当 *Call Stack* 清空时（同步代码执行完了, 逻辑循环结束之后）
- 触发**Event Loop**机制 （不断观察）
- 轮询查找*Callback Queue*，如果有回调方法则放入*Call stack* 中执行
- 然后继续轮询（像永动机一样）

| 核心组件       | 作用                   | 优先级/调度                                                              | 示例                                          |     |
| -------------- | ---------------------- | ------------------------------------------------------------------------ | --------------------------------------------- | --- |
| *Call Stack*   | 执行同步代码           | 最高，必须清空                                                           | `console.log`、`for()`                        |     |
| *Web APIs*     | 处理异步代码的等待时间 | 不参与调度，只计时/等待                                                  | `setTimeout`、`fetch()`、`addEventListener()` |     |
| *Micro-Task*   | 存放微任务的回调函数   | 次高，同步代码执行完成后执行微任务，在每个宏任务执行完之后也会清空微任务 | `promise`、`async/await`                      |     |
| *Macro-Task*   | 存放宏任务的回调函数   | 最低，每个循环只取一个执行                                               | `setTimeout`、`setImmediate`、`I/O`、UI渲染   |     |
| **Event Loop** | 驱动机制               | 不断检查*Call Stack*状态，驱动任务进入                                   | 永远执行                                      |     |

---
### 微任务 + DOM的情况下
1. *Call Stack*清空
2. 执行当前的微任务， *Macro-Task*
3. 尝试*DOM*渲染
4. 触发**Event Loop** 机制

---
## Node.js环境（浅说）

**Event Loop**分为几个阶段
1. *Timers*阶段 （处理`setTimeout`、`setInterval`）
2. *Poll*阶段 （处理IO、网络服务）
3. *Check*阶段 （处理`setImmdiate`）