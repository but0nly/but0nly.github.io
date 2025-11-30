---
title: Webpack的原理
date: 2023-10-08
tags: 
 - 前端
 - 工程化
categories: 前端
keywords: 
 - Webpack
---

### 作用
**webpack**是一个打包器，主要目标将多种模块打包成浏览库可以加载执行的文件。


### 核心概念
1. *entry*（入口）
2. *Dependency Graph*（依赖图）
3. *output*（输出）
4. *loaders*（加载器）
5. *plugins*（插件）

### 打包流程
1. **初始化**（读取配置，创建*Compiler*）
	1. 读取配置：加载（`webpack.config.json`）文件中的配置（Entry\output\loaders\plugins等）
	2. 创建*Compiler*实例：是Webpack的一个核心对象，包含了整个打包环境和配置信息。
2. **编译&模块构建**（*Compiler*启动*Compilation*，从**Entry**开始编译）
	1. 生成`Compilation`编译对象：由 Compiler 创建，它负责**执行**一次完整的打包会话。它包含了当前打包过程中的所有信息，如模块、依赖关系、资源块（Chunks）等等
	2. Compilation 实例从配置的*Entry*入口开始。
	3. 将*Entry*文件视为一个模块，然后利用**Loaders**来处理这个文件内容。
	4. 在处理过程中，webpack会解析文件中所有的`import`、`require`
3. **模块解析**（确定所有依赖的绝对路径）
	1. 在模块中遇到`import "react"`或者`import "./utils.js"`，就会通过*解析机制*来确定文件的物理文件路径在哪。
	2. 配合**加载器**转换，并放到**依赖图**中
4. **分块**
	1. 将这些模块（Chunk）组织成文件
5. **优化&代码分割**
	1. 优化：`tree-shaking`、`minification`
	2. 代码分割：
		1. 按需加载：`import()`
		2. 缓存优化：将第三方库分割成`vendor chunk`，方便浏览器存储。
6. **资源生成&写入**
	1. 资源生成：Compilation 将内存中优化好的 chunks 映射成一个或多个*Assets*
	2. 写入：根据`webpack`中配置的output规则，将这些文件写入到系统中。

#### 总结：
1. 读取配置，创建*Compiler*。
2. *Comiler*启动*Compilation*，从**Entry**开始。
3. 模块解析：确定所有依赖的绝对路径。
4. 加载器转换：使用**Loaders**处理非JS文件。
5. 分块和优化：将模块组织成==Chunks==，并进行 *Tree Shaking* 优化。
6. 资源写入：根据**Output**配置生成并写入到系统中。