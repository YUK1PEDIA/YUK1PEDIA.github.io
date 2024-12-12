---
layout: post
title: IntelliJ IDEA 中 Lombok 注解失效问题
description: 记录
tag: 杂七杂八
---

## 问题

尝试在 IDEA 中使用 Lombok 注解，但是在程序编译时出现找不到 `get&set` 方法、找不到 `builder` 方法等报错，但在对应的类中都已经加上了 `@Data` 和 `@Builder` 注解，但是编译器找不到注解。



## 解决方法

确保安装了 Lombok 插件，（`Settings` → `Plugins` 安装 `lombok`）

![1.png](https://s2.loli.net/2024/12/11/3TvxmLraowtRcnd.png)

并且启用了注解处理器（`Settings` → `Build, Execution, Deployment` → `Compiler` → `Annotation Processors`）

![1.png](https://s2.loli.net/2024/12/11/mNXBcZTjpYCOtfM.png)

做完上述两步后即可通过编译