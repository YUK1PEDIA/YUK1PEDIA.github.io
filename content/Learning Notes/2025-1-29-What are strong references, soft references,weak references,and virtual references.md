---
layout: post
title: 什么是强引用、软引用、弱引用和虚引用？
description: 记录
tag: 笔记
---

## 1. 强引用（Strong Reference）

**强引用**是最普通的引用，比如 `Object obj = new Object();`。只要强引用存在，对象就**不会被垃圾回收**。

**强引用特点**：

- **绝不回收**：只要强引用存在，即使内存不足（OOM），JVM 宁可抛出内存错误，也不回收对象。
- **常见场景**：日常代码中的普通对象都是强引用。



## 2. 软引用（Soft References）

**软引用**通过 `SoftReference` 类实现，比如 `SoftReference<Object> softRef = new SoftReference<>(obj);` 。

**软引用特点**：

- **内存不足时回收**：当 JVM 发现内存不足时，会回收软引用指向的对象。
- **适合做缓存**：比如缓存图片，内存不足时自动清理，避免 OOM 。

```java
// 创建一个软引用
SoftReference<byte[]> softRef = new SoftReference<>(new byte[1024 * 1024]);
// 使用对象
byte[] data = softRef.get(); // 如果对象未被回收，可以拿到
```



## 3. 弱引用（Weak Reference）

**弱引用**通过 `WeakReference` 类实现，比如 `WeakReference<Object> weakRef = new WeakReference<>(obj);` 。

**弱引用特点**：

- **立即回收**：和软引用类似，但软引用在发生垃圾回收时**不一定会被回收**（内存不足时软引用才被回收），而弱引用无论内存是否充足，只要发生垃圾回收，弱引用指向的对象都会被回收。
- **适合临时缓存**：比如维护一种可有可无的缓存（如 `ThreadLocal` 中的 Entry 就是弱引用）。

```java
WeakReference<Object> weakRef = new WeakReference<>(new Object());
System.gc(); // 触发垃圾回收（非强制）
System.out.println(weakRef.get()); // 可能输出 null
```



## 4. 虚引用（Phantom Reference）

**虚引用**通过 `PhantomReference` 类实现，必须配合**引用队列**使用。举个例子，虚引用就像在物品（对象）上安装了一个“销毁报警器”。当物品被销毁时（对象被垃圾回收），你会收到一个通知，但是你永远拿不到这个物品本身。

**虚引用特点**：

- **无法获取对象**：虚引用的 `get()` 方法永远返回 `null` 。
- **仅用于跟踪回收**：唯一作用是监听对象何时被回收（通过引用队列）。
- **管理特殊资源**：比如回收时触发某些操作（如释放堆外内存）。

```java
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantomRef = new PhantomReference<>(new Object(), queue);
// 当对象被回收时，phantomRef 会被加入队列
```



## 补充：虚引用是如何进行底层资源管理的？

虚引用主要通过监听对象回收事件，触发一些必须在对象被销毁后执行的清理操作（比如释放堆外内存、关闭文件句柄等）。它的核心逻辑是：**即使对象已经被垃圾回收，仍能通过虚引用感知这一事件，并执行后需清理**。

以**堆外内存管理**为例：

- Java 的 `DirectByteBuffer` 对象内部会分配一块堆外内存（通过 `malloc` 或 `Unsafe.allocateMemory`），这部分内存不受 JVM 垃圾回收管理。
- 当 `DirectByteBuffer` 对象被回收时，堆外内存不会自动释放，必须手动调用 `free` 方法。
- **问题**：无法直接知道 `DirectByteBuffer` 何时被回收，可能导致堆外内存泄漏。
- **解决方案**：用虚引用监听 `DirectByteBuffer` 的回收事件，触发堆外内存的释放。

虚引用的工作流程：

1. 创建虚引用并绑定引用队列：

   ```java
   ReferenceQueue<MyResource> queue = new ReferenceQueue<>();
   PhantomReference<MyResource> phantomRef = new PhantomReference<>(new MyResource(), queue);
   ```

   - `MyResource` 是需要管理资源的对象（例如 `DirectByteBuffer`）
   - 当 `MyResource` 对象被垃圾回收时，`phantomRef` 会被加入 `queue`

2. 后台线程监听引用队列：

   ```java
   new Thread(() -> {
       while (true) {
           PhantomReference<?> ref = (PhantomReference<?>) queue.poll();
           if (ref != null) {
               // 执行资源清理操作（例如释放堆外内存）
               cleanupResource(ref);
           }
       }
   }).start();
   ```

   - 通过轮询队列，可以及时获取到被回收的对象对应的虚引用

3. 资源清理：

   - 在 `cleanupResource` 方法中，可以通过虚引用关联的元数据（如内存地址）执行实际清理操作