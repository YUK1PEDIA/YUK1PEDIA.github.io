---
layout: post
title: JavaGuide
description: 记录
tag: 笔记
---

# JavaGuide

## 前言

自用，整理自 JavaGuide 八股（仍在学习），原文地址：[Java 面试指南 | JavaGuide](https://javaguide.cn/)



## 基础

### 基本数据类型

#### Java中的几种基本数据类型了解么？

Java中有 8 种基本数据类型：

- 6 种数字类型
  - 4 种整数型：`byte`、`short`、`int`、`long`
  - 2 种浮点型：`float`、`double`
- 1 种字符类型：`char`
- 1 种布尔型：`boolean`

这 8 种基本数据类型的默认值以及所占空间的大小如下：

| 基本类型  | 位数 | 字节 | 默认值  | 取值范围                                                     |
| :-------- | :--- | :--- | :------ | ------------------------------------------------------------ |
| `byte`    | 8    | 1    | 0       | -128 ~ 127                                                   |
| `short`   | 16   | 2    | 0       | -32768（-2^15） ~ 32767（2^15 - 1）                          |
| `int`     | 32   | 4    | 0       | -2147483648 ~ 2147483647                                     |
| `long`    | 64   | 8    | 0L      | -9223372036854775808（-2^63） ~ 9223372036854775807（2^63 -1） |
| `char`    | 16   | 2    | 'u0000' | 0 ~ 65535（2^16 - 1）                                        |
| `float`   | 32   | 4    | 0f      | 1.4E-45 ~ 3.4028235E38                                       |
| `double`  | 64   | 8    | 0d      | 4.9E-324 ~ 1.7976931348623157E308                            |
| `boolean` | 1    |      | false   | true、false                                                  |

Java 的每种基本类型所占存储空间的大小不会像其他大多数语言那样随机器硬件架构的变化而变化。**这种所占存储空间大小的不变性**是Java 程序比用其他大多数语言编写的程序更具**可移植性**的原因之一。

注意：

1. Java 里使用 `long` 类型的数据一定要在数值后面加上 **L**，否则将作为整型解析。

2. Java 里使用 `float` 类型的数据一定要在数值后面加上 **f 或 F**，否则将无法通过编译。

3. `char a = 'h'`char :单引号，`String a = "hello"` :双引号。

这八种基本类型都有对应的包装类分别为：`Byte`、`Short`、`Integer`、`Long`、`Float`、`Double`、`Character`、`Boolean`。





#### 基本类型和包装类型的区别？

- **用途**：除了定义一些常量和局部变量之外，我们在其他地方比如**方法参数、对象属性**中很少会使用基本类型来定义变量。并且，**包装类型可用于泛型**，而基本类型不可以。
- **存储方式**：基本数据类型的**局部变量**存放在 Java 虚拟机**栈**中的局部变量表中，基本数据类型的**成员变量**（未被 `static` 修饰 ）存放在 Java 虚拟机的**堆**中。**包装类型属于对象类型**，我们知道几乎所有对象实例都存在于堆中。

- **占用空间**：相比于包装类型（对象类型）， 基本数据类型占用的空间往往非常小。
- **默认值**：成员变量包装类型不赋值就是 `null` ，而基本类型有默认值且不是 `null`。
- **比较方式**：对于基本数据类型来说，`==` 比较的是值。对于包装数据类型来说，`==` 比较的是**对象的内存地址**。所有整型包装类对象之间**值的比较**，全部使用 `equals()` 方法。

⚠️ 注意：**基本数据类型存放在栈中是一个常见的误区！** 基本数据类型的存储位置取决于它们的**作用域和声明方式**。如果它们是**局部变量**，那么它们会存放在**栈**中；如果它们是**成员变量**，那么它们会存放在**堆**中。

```java
public class Test {
    // 成员变量，存放在堆中
    int a = 10;
    // 被 static 修饰，也存放在堆中，但属于类，不属于对象
    // JDK1.7 静态变量从永久代移动了 Java 堆中
    static int b = 20;

    public void method() {
        // 局部变量，存放在栈中
        int c = 30;
        static int d = 40; // 编译错误，不能在方法中使用 static 修饰局部变量
    }
}
```





#### 包装类型的缓存机制

在 Java 中，包装类型的缓存机制是一种优化策略，用于提高性能和减少内存使用。通过缓存常用的包装类型对象，可以**避免在频繁创建相同值的对象时重复分配内存**。

包装类型的缓存机制是 Java 针对一些特定的包装类（如 `Integer`、`Short`、`Byte`、`Character` 和 `Boolean`）实现的一种优化。它会为某些常用的值预先创建对象并缓存起来，当需要这些值的包装对象时，直接从缓存中返回现有的对象，而不是每次都创建一个新的对象。这有助于提高性能并节省内存。

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b); // 输出 true，因为 a 和 b 指向同一个缓存对象
```

`Byte`,`Short`,`Integer`,`Long` 这 4 种包装类默认创建了数值 **[-128，127]** 的相应类型的缓存数据，`Character` 创建了数值在 **[0,127]** 范围的缓存数据，`Boolean` 直接返回 `True` or `False`。



**Integer 缓存源码**

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static {
        // high value may be configured by property
        int h = 127;
    }
}
```



**Character 缓存源码**

```java
public static Character valueOf(char c) {
    if (c <= 127) { // must cache
      return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}

private static class CharacterCache {
    private CharacterCache(){}
    static final Character cache[] = new Character[127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i)
    }
}
```



**Boolean 缓存源码**

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```



**注意：两种浮点数类型的包装类 `Float`,`Double` 并没有实现缓存机制。**

下面我们来看一个问题：下面的代码的输出结果是 `true` 还是 `false` 呢？

```java
Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2);
```

`Integer i1 = 40` 这一行代码会发生**装箱**，也就是说这行代码等价于 `Integer i1 = Integer.valueOf(40)` 。因此，`i1` 直接使用的是**缓存中的对象**。而`Integer i2 = new Integer(40)` 会直接**创建新的对象**。因此输出结果是 `false`



记住：**所有整型包装类对象之间值的比较，全部使用 equals 方法比较**。

![img](https://oss.javaguide.cn/github/javaguide/up-1ae0425ce8646adfb768b5374951eeb820d.png)



#### 自动装箱和拆箱了解吗？原理是什么？

**什么是自动拆装箱？**

- **装箱**：将基本类型用它们对应的引用类型包装起来；
- **拆箱**：将包装类型转换为基本数据类型；

```java
Integer i = 10;  //装箱
int n = i;   //拆箱
```

以上两行代码的字节码如下：

```java
   L1

    LINENUMBER 8 L1

    ALOAD 0

    BIPUSH 10

    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;

    PUTFIELD AutoBoxTest.i : Ljava/lang/Integer;

   L2

    LINENUMBER 9 L2

    ALOAD 0

    ALOAD 0

    GETFIELD AutoBoxTest.i : Ljava/lang/Integer;

    INVOKEVIRTUAL java/lang/Integer.intValue ()I

    PUTFIELD AutoBoxTest.n : I

    RETURN
```

从字节码中，我们发现**装箱**其实就是**调用了包装类的`valueOf()`方法**，**拆箱**其实就是**调用了 `xxxValue()`方法**。

因此，

- `Integer i = 10` 等价于 `Integer i = Integer.valueOf(10)`
- `int n = i` 等价于 `int n = i.intValue()`



注意：**如果频繁拆装箱的话，也会严重影响系统的性能。我们应该尽量避免不必要的拆装箱操作。**

```java
private static long sum() {
    // 应该使用 long 而不是 Long
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

