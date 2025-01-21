---
layout: post
title: 死锁产生的条件？如何诊断死锁？
description: 记录
tag: 笔记
---

## 死锁产生的条件

在操作系统中，死锁只有**同时满足以下四个条件**才会发生：

- 互斥条件
- 持有并等待条件
- 不可剥夺条件
- 环路等待条件

在 java 程序中，由于 java 自带线程，因此自己编写的两条线程也可能出现死锁的情况。

具体而言，java 程序里死锁产生的条件是两条线程分别持有两个锁，并且都在尝试获取对方所持有的锁，这样就造成了死锁。

以下面的程序为例：

```java
public class Main {
    public static void main(String[] args) {
        Object A = new Object();
        Object B = new Object();
        Thread t1 = new Thread(() -> {
            synchronized (A) {
                System.out.println("lock A");
                try {
                    sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                synchronized (B) {
                    System.out.println("lock B");
                    System.out.println("操作...");
                }
            }
        }, "t1");

        Thread t2 = new Thread(() -> {
            synchronized (B) {
                System.out.println("lock B");
                try {
                    sleep(500);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                synchronized (A) {
                    System.out.println("lock A");
                    System.out.println("操作...");
                }
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}
```

线程 t1 和线程 t2 分别获取了锁 A 和锁 B ，并且都在尝试获取对方的锁，因此就出现了死锁：

![image.png](https://s2.loli.net/2025/01/21/aTIVAX5tdpxw18q.png)





## 如何诊断死锁

使用 JDK 自带的工具：jps 和 jstack

- jps：输出 JVM 中运行的进程状态信息
- jstack：查看 java 进程内线程的堆栈信息，查看日志，检查是否有死锁，如果有死锁现象，需要查看具体代码分析后修复

在运行上述代码后出现了死锁，这里可以在 terminal 输入上面两条命令来检测哪里出现了死锁：

1. 输入 jps 查看当前正在运行的线程

![image.png](https://s2.loli.net/2025/01/21/CDT5Z2YmuV6XNKW.png)

2. 使用 jstack 查看哪里出现了死锁，根据堆栈信息进行 bug 定位并排查即可

![image.png](https://s2.loli.net/2025/01/21/3RkxoZ9d8HeNSal.png)

