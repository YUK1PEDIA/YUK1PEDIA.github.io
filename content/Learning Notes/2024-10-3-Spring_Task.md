---
layout: post
title: Spring Task
description: 记录
tag: 笔记
---

## 什么是 Spring Task

Spring Task 是 Spring 框架提供的任务调度工具，可以按照约定的时间自动执行某个代码逻辑，是一个**定时任务框架**。

应用场景：

- 信用卡每月还款提醒
- 银行贷款每月还款提醒
- 火车票售票系统处理未支付订单

……





## cron 表达式

通过 cron 表达式可以**定义任务触发的事件**

构成规则：分为 6 或 7 个域，由空格分割开，每个域代表一个含义

每个域的含义分别为：秒、分钟、小时、日、月、周、年（可选）。需要注意的是，**周**和**日**两者只能填一个，周代表是周几，日代表是这个月的多少号。

| 秒   | 分钟 | 小时 | 日   | 月   | 周   | 年   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 0    | 0    | 9    | 12   | 10   | ?    | 2024 |

2024年10月12日上午9点整对应的 cron 表达式为：`0 0 9 12 10 ? 2024`

实际上 cron 表达式不需要我们去手写，可以使用在线生成器生成：[在线Cron表达式生成器 (qqe2.com)](https://cron.qqe2.com/)





## Spring Task 使用步骤

1. 导入 maven 坐标 spring-context
2. 启动类添加注解 `@EnableScheduling` 开启任务调度

```java
package com.sky;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@SpringBootApplication
@EnableTransactionManagement //开启注解方式的事务管理
@Slf4j
@EnableCaching // 开启缓存注解
@EnableScheduling // 开启任务调度
public class SkyApplication {
    public static void main(String[] args) {
        SpringApplication.run(SkyApplication.class, args);
        log.info("server started");
    }
}
```

3. 自定义定时任务类

```java
package com.sky.task;

import lombok.extern.slf4j.Slf4j;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * 自定义定时任务类
 */
@Component
@Slf4j
public class MyTask {

    /**
     * 定时任务，每隔五秒触发一次
     */
    @Scheduled(cron = "0/5 * * * * ?")
    public void executeTask() {
        log.info("定时任务开始执行：{}", new Date());
    }
}
```

