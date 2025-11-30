---
title: Promise经典面试题
date: 2023-10-02
tags:
 - 前端
 - JavaScript
categories: 前端
keywords: 
 - Promise
---

题目:
```js
Promise.resolve().then(()=>{
	console.log(0);
	return Promise.resolve(4)
}).then((res)=>{
	console.log(res);
})

Promise.resolve().then(()=>{
	console.log(1);
}).then(()=>{
	console.log(2);
}).then(()=>{
	console.log(3);
}).then(()=>{
	console.log(5);
})
```
答案：0 1 2 3 4 5；


## 分析
---

### then 的交替执行

- 如果有多个 `fulfilled`的**Promise**实例，同时执行`then`链式调用
- `then`会交替执行
- 这是编译器的优化，防止一个**Promise**占据太多时间


### then中返回promise实例

- 相当于多出了一个**Promise**实例
- 也会遵循“交替执行”
- 它和直接申明一个**Promise**实例，结果有些差异
	- *会慢两拍*


### 慢两拍

- 第一拍：等待`pending`状态变成`fulfilled`状态
- 第二拍：`then`函数挂载到*MicroTaskQueue（微任务队列）*

以上面代码演示：
```js
Promise.resolve().then(()=>{
	// 第一拍 等待pending状态变成fulfill状态
	const p = Promise.resolve(0);
	
	Promise.resolve().then(()=>{
		// 第二拍 then函数挂载到微任务队列
		p.then((res)=>{
			console.log(res);
		}).then(()=>{
			console.log(1)
		}).then(()=>{
			console.log(2)
		}).then(()=>{
			console.log(3)
		}).then(()=>{
			console.log(4)
		})
	})
})
```


### 分析结果

- 输出0
- *交替执行*
- 输出1
- 遇到`then`返回**Promise**实例
- *慢第一拍*
- 输出2
- *慢第二拍*
- 输出3
- `then`挂载完毕
- *交替执行*
- 输出4
- *交替执行*
- 输出5


## 总结
---

- Event Loop、宏任务、微任务
- `then`交替执行
- `then`返回**Promise**实例会慢两拍