---
title: 客户端 Mock 解决方案
date: 2024-02-13
tags: 
  - 前端
categories: 团队协作
keywords:
  - Mock
---

# 轻量级浏览器端网络 Mock 工具技术方案

## 1\. 项目背景

### 1.1 现状与痛点

在前后端分离的开发模式中，前端开发往往依赖后端 API 的完成度。当后端接口尚未就绪或需要模拟特定异常场景（如 500 错误、超时、特定数据结构）时，开发人员通常面临以下困境：

  * **硬编码 Mock：** 在业务代码中写入 Mock 数据，导致代码污染，上线前需手动清理，风险较高。
  * **重型工具依赖：** 使用 Charles/Fiddler 等抓包工具进行 Map Local，配置繁琐，且难以在团队间共享规则。
  * **本地 Node 服务器：** 需要维护额外的 Mock Server，增加了开发环境的复杂度。

### 1.2 目标

构建一个**零侵入、可视化、轻量级**的浏览器端 Mock 工具。

  * **零侵入：** 无需修改业务代码，通过脚本注入（Tampermonkey）即可运行。
  * **可视化：** 提供嵌入页面的 GUI 面板，实时查看请求并编辑响应。
  * **隔离性：** 利用 Shadow DOM 确保工具样式不污染业务页面。

-----

## 2\. 技术方案

### 2.1 核心架构

本方案采用 **Web Components** 构建 UI，配合 **Monkey Patching (猴子补丁)** 技术拦截浏览器网络请求。数据持久化依赖浏览器的 `localStorage`。

| 模块         | 技术选型                                          | 说明                                                                     |
| :----------- | :------------------------------------------------ | :----------------------------------------------------------------------- |
| **UI 视图**  | **Web Components (Custom Elements + Shadow DOM)** | 实现样式完全隔离，避免 CSS 冲突；原生支持，无框架依赖。                  |
| **请求拦截** | **Monkey Patching (`window.fetch` 重写)**         | 在 Tampermonkey 环境下，直接覆盖原生 Fetch API，实现请求的“中间人”拦截。 |
| **数据存储** | **`localStorage`**                                | 存储 Mock 规则、面板位置及状态。支持按域名（Host）隔离存储。             |
| **脚本注入** | **Tampermonkey (油猴脚本)**                       | 作为载体，将工具注入到目标网页的 `document-start` 阶段。                 |

### 2.2 逻辑流程

1.  **初始化：** 脚本在页面加载最早阶段运行，覆盖 `window.fetch`，并挂载 Web Component 面板。
2.  **请求发起：** 业务代码调用 `fetch`。
3.  **拦截判断：** 拦截器归一化请求参数，查询 Mock 规则存储。
      * **命中规则：** 阻止真实网络请求，构造自定义 `Response` 对象返回（状态码、响应体、Header）。
      * **未命中规则：** 放行真实请求，但劫持其响应流（Clone Response）以便在面板中记录日志。
4.  **UI 交互：** 用户在面板中查看日志，勾选请求进行 Mock 编辑，保存后即时生效。

-----

## 3\. 详细设计与实现

### 3.1 模块划分

#### A. `MockStorage` (存储层)

负责数据的读写操作，核心特性是**多域名隔离**。

  * **数据结构设计：**
    ```json
    {
      "mockRules": {
        "dev.example.com": { "/api/user": { "enabled": true, "responseBody": "..." } },
        "test.example.com": { "/api/list": { "enabled": false, "responseBody": "..." } }
      },
      "mockPanelPosition": { "top": "100px", "left": "200px" },
      "mockPanelIsMinimized": "false"
    }
    ```
  * **API：** `getRules()`, `setRule()`, `deleteRule()`, `saveRules()`。所有读写操作自动根据 `window.location.host` 进行 key 隔离。

#### B. `NetworkMockPanel` (视图层)

基于 `HTMLElement` 的自定义组件。

  * **Shadow DOM：** 内部封装 `<style>` 和 HTML 模板，确保工具 UI 不受外部 CSS 影响（如全局 `box-sizing` 或 `reset.css`）。
  * **布局：** 采用 Flex 布局。左侧为请求列表（支持筛选），右侧为详情/编辑器（支持 Tab 切换）。
  * **交互特性：**
      * **拖拽：** 监听 `mousedown/mousemove/mouseup`，支持全屏拖拽。
      * **最小化/恢复：** 记录尺寸，切换显示模式，并更新按钮图标（`_` vs `□`）。
      * **Toast 通知：** 替代 `alert`，在 Shadow DOM 内实现自毁式轻提醒。

#### C. `Interceptor` (拦截层)

  * **参数归一化：** 处理 `fetch(url)` 和 `fetch(Request)` 两种调用方式，统一提取 URL 字符串。
  * **规则匹配：** 支持精确匹配和通配符匹配（如 `/api/*`）。
  * **响应伪造：** 使用 `new Response(body, init)` 构造符合 Fetch 标准的返回值。

### 3.2 关键代码实现摘要

**请求拦截核心逻辑：**

```javascript
const originalFetch = window.fetch;
window.fetch = function (input, init) {
    // 1. 参数处理
    let url = input instanceof Request ? input.url : String(input);
    
    // 2. 规则匹配
    const rules = MockStorage.getRules(); // 获取当前域名的规则
    const mockRule = findMatchingMockRule(url, rules);

    // 3. 命中 Mock
    if (mockRule && mockRule.enabled) {
        // 返回伪造的 Promise
        return Promise.resolve(new Response(mockRule.responseBody, {
            status: mockRule.statusCode,
            headers: { 'X-Mocked-By': 'Tampermonkey' }
        }));
    }

    // 4. 未命中，走真实请求并记录日志
    return originalFetch.apply(this, arguments).then(res => {
        // Clone 响应以避免流被锁死
        res.clone().text().then(body => logToPanel(url, body));
        return res;
    });
};
```

-----

## 4\. 核心功能与优化

基于开发过程中的迭代，本工具包含以下高级特性：

### 4.1 域名隔离支持

  * **背景：** 不同环境（Dev/Test/Prod）或不同项目的域名不同，共用一套规则会导致冲突。
  * **实现：** `MockStorage` 在存取时自动追加 `window.location.host` 作为命名空间。

### 4.2 URL 智能筛选

  * **功能：** 请求列表顶部提供输入框。
  * **实现：** 监听 `input` 事件，实时遍历 DOM 列表，通过 `row.style.display` 控制显隐，解决接口过多难以查找的问题。

### 4.3 状态持久化

  * **面板位置：** 拖拽结束时记录 `top/left` 坐标，刷新后自动还原位置。
  * **最小化状态：** 记录面板是否被收起。刷新页面后，若之前是最小化状态，工具会自动初始化并立即执行最小化逻辑。

### 4.4 交互体验优化

  * **Toast 轻提醒：** 封装 `showToast` 方法，提供 Success/Error 状态的非阻塞式通知。
  * **防误触拖拽：** 在 Header 拖拽逻辑中，增加 `target` 检查。若点击的是 Header 内部的按钮（关闭/最小化），则通过 `while` 循环向上查找并拦截，防止触发拖拽。
  * **长 URL 适配：** 编辑器头部采用 `flex-wrap: wrap` 和 `word-break: break-all`，确保超长 URL 能够完整展示且不挤压操作按钮。

### 4.5 规则管理

  * **单条删除：** 编辑器内提供删除按钮，删除特定 URL 规则。
  * **一键清空：** 头部提供垃圾桶图标，清空当前域名下所有规则（带二次确认）。

-----

## 5\. 最后
附上我的仓库地址:
油猴脚本版: [Mock-Network-Panel](https://github.com/but0nly/scripts/blob/master/src/mock-network-panel.js)
npm包版: [Mock-Network-Tool](https://github.com/but0nly/Mock-Network-Tool)
