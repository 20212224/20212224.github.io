---
title: bug排查01
date: 2025-05-17 18:03:46
tags: vue
---

最近在使用 **Vue 3** 和 **Pinia** 开发日历待办插件时，数据流和响应式更新机制出现冲突，导致视图不更新。

* **数据源:** Pinia Store 中的 state (通常是 Proxy 对象)。
* **流程:** Pinia State 数据 $\rightarrow$ 通过方法暴露给待办设置窗口 $\rightarrow$ 设置并保存 $\rightarrow$ 返回数据 $\rightarrow$ 日历绘制页接收并赋值给本地 `ref()` 变量。
* **症状:** 尽管本地 `ref` 变量已赋值，但页面视图**未更新**。

问题出在 Vue 3 的 `ref()` 机制和 Pinia Store 返回的 **Proxy/响应式对象**的交互上。

**Vue 3 `ref` 的通知机制：**

一个 `ref` **仅**在它的内部值 (`.value`) 的**引用地址**发生变化时，才会通知依赖（即组件视图）进行更新。

* **赋值操作：** 当您执行 `myRef.value = newValue` 时，Vue 3 内部会进行 `Object.is(myRef.value, newValue)` 引用地址比较。

**从proxy取出对象会自动包装成proxy**

后面在赋值时进行了深拷贝解决了这个问题.