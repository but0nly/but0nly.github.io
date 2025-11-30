---
title: Webpack的执行流程
date: 2023-10-08
tags: 
 - 前端
 - 工程化
categories: 前端
keywords: 
 - Webpack
---

**webpack**启动后会在entry配置的module开始递归解析 每找到一个module，就会根据配置loader的去找对应的转换规则，对module进行转换后开始解析module所依赖的module，然后这些模块以entry分组，一个entry和所有相依赖的模块对应一个chunk，最后webpack会把有所有的chunk转换成文件输出，在整个流程中，webpack会在恰当的时机执行plugin的逻辑。


1. 会从 *entry* 第一个 `module` 开始递归解析
2. 每找到一个 `module` 就会根据配置的*loader*规则转换
3. 转换后开始解析 `module` 所依赖的 `module` 
4. 然后这些 `module` 以 *entry* 分组，一个 *entry* 所依赖的模块对应一个 ==chunk==
5. **webpack** 会把这些 ==chunk== 转换成文件根据 *output* 输出
6. 整个流程中 *plugins* 会在适当的时机执行