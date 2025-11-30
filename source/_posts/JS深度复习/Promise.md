---
title: Promise
date: 2023-10-02
tags:
 - 前端
 - JavaScript
categories: 前端
keywords:
 - Promise
---

### 概念与定义

- **Promise** 是一个对象
	- 代表了一个未来才知道的值
- **Promise** 诞生的目的是为了解决回调地狱（*Callback Hell*）的问题，让异步代码更好的组织和阅读。

> 特性： 链式调用， 将代码扁平化，更好的可读性和可维护性。

### **Promise**的三种状态

- `pending`（等待）： 初始状态，异步操作正在进行中，既没有*成功*也没有*失败*；
- `fulfilled`（已成功/已解决）：操作成功，此时会有一个结果值(*value*);
- `rejected`(已失败/已拒绝)：操作失败，此时会有一个失败原因（reason，通常为*error*）

> 一旦**Promise**状态从`pending` 变为`fulfilled`或`rejected`，它就处于`settled`，并且状态不可逆转。


### **Promise**的实例方法

| 方法                               | 用途                                                                                                                 |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `.then(onFulfilled, [onRejected])` | 用于处理`fulfilled`状态的结果（第一个参数）和处理`rejected`状态的结果（第二个参数，可选）                            |
| `.catch(onRejected)`               | 专门处理`rejected`状态的错误。                                                                                       |
| `.finallya(onSettled)`             | 无论**Promise**最终返回的是`fulfilled`还是`rejected`，都会执行的回调。它的回调函数不接受任何参数，只负责作清理工作。 |
### Promise的静态方法

| 方法                           | 用途                                                                                                                                                           | 应用场景                                                                 |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `Promise.all(iterable)`        | 1. 等待所有**Promise**完成（成功或失败）<br>2. 成功条件：所有的**Promise**都成功`fulfilled`<br>3. 失败条件：只要有一个**Promise**返回`rejected`                | 同时加载多个资源，等待所有资源都加载完成后再进行下一步                   |
| `Promise.race(iterable)`       | 1. 等待最快的**Promise**完成<br>2. 成功/失败条件：第一个完成（无论成功还是失败）的**Promise**，其状态和结果都作为`Promise.race()`的状态和结果                  | 限制响应时间                                                             |
| `Promise.allSettled(iterable)` | 1. 等待所有的**Promise**都敲定（`settled`）<br>2. 成功条件：所有的**Promise**都敲定了（无论成功或失败）<br>3. 失败条件：永远不会失败<br>                       | 执行多个不相关的异步操作，并且无论它们成功与否，都需要知道它们的执行结果 |
| `Promise.any(iterable)`        | 1. 等待第一个成功返回的**Promise**<br>2. 成功条件：只要有一个**Promise**成功`fulfilled`<br>3. 失败条件：所有的**Promise**都失败`rejected`                      | 从多个镜像或着数据源中获取*数据*，只要有一个成功返回即可                 |
| `Promise.resolve(value)`       | 1. 快速创建一个*成功*的**Promise**<br>2. 返回：一个状态为`fulfilled`的**Promise**，并带有指定的值(`value`)<br>3. 如果本身就是一个**Promise**，就直接返回本身。 | 将一个普通值封装成一个**Promse**，以便在**Promise**链中调用。            |
| `Promise.reject(reason)`       | 1. 快速创建一个*失败*的**Promise**<br>2. 返回：一个状态为`rejected`的**Promise**，并带有其失败原因                                                             | 在某些逻辑分支中，需要立即返回一个失败的 Promise。                       |
