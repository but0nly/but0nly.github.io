---
title: 关于使用 IndexDB 的坑
date: 2025-01-18
tags: 
  - 前端
  - 缓存
categories: 前端
keyworks:
  - IndexDB
  - 缓存
---

由于IndexDB 如果频繁更新的话会导致数据丢失，跟SQL一样。

### 解决方案
内存缓存 + 节流 + 批量写入

1. 创建一个JS对象，用于存储每次操作的`key`和`value`；
2. 创建一个`autoSave`方法，里面就是通过创建`readwrite`事务，然后`put`新的配置信息。
3. 给`autoSave`添加节流，每次触发1s后执行。
4. 创建一个JS赋值函数，附带监听JS对象的赋值是否存在==变化==
5. 如果存在变化就`autoSave`
6. 完美解决批量更新导致数据丢失的*Bug*
