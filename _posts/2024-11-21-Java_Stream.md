---
layout: post
title: Java Stream
description: 记录
tag: 笔记
---

## 简介

Java 的 `Stream` API 是 Java 8 引入的一项功能，它主要用于处理集合类（如 `List`、`Set`、`Map` 等）中的元素，提供了一种声明性的方法来进行数据处理。`Stream` API 提供了许多可以连缀使用的操作，支持高效的顺序和并行流处理。

**Stream** 并不是存储数据的结构，而是对集合、数组或其他数据源进行操作的一种方式。Stream 对象可以从集合、数组、文件等多种数据源获取。



## Stream 的基本特征

1. **惰性求值**（Lazy Evaluation）：Stream 的操作是懒惰求值的，只有在最终操作被触发时，Stream 才会进行计算。
2. **可以组合操作**：Stream 允许将多个操作链式组合，从而形成一条清晰的流水线。
3. **支持并行处理**：Stream 支持通过并行流（`parallelStream()`）来并发处理数据，能够充分利用多核 CPU，提高处理效率。
4. **无副作用**：Stream 的操作通常是无副作用的，即不会改变原始数据源。



## Stream 的操作分类

Stream 的操作分为两种类型：

- **中间操作**：如 `map()`、`filter()`、`sorted()`、`distinct()` 等。中间操作是惰性求值的，只有在终结操作（如 `collect()`、`forEach()`）触发时，才会开始执行。
- **终结操作**：如 `collect()`、`forEach()`、`reduce()`、`count()` 等。终结操作触发整个流水线的执行，并且一次执行后，Stream 就会被消费掉，**不能再重新使用**。



### 常用的中间操作

1. `filter()`

   - 功能：通过给定的条件过滤流中的元素，返回符合条件的元素

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
     List<Integer> evenNumbers = numbers.stream()
                                        .filter(n -> n % 2 == 0)
                                        .collect(Collectors.toList());
     System.out.println(evenNumbers); // [2, 4, 6]
     ```

2. `map()`

   - 功能：对流中的**每个元素**执行一个函数，返回一个新的流（元素类型可以改变）

   - 示例：

     ```java
     List<String> words = Arrays.asList("apple", "banana", "cherry");
     List<String> upperWords = words.stream()
                                    .map(String::toUpperCase)
                                    .collect(Collectors.toList());
     System.out.println(upperWords); // [APPLE, BANANA, CHERRY]
     ```

3. `flatMap()`

   - 功能：将每个元素转换为一个流，并将所有的流扁平化为一个单一的流**（可以用于处理嵌套集合）**

   - 示例：

     ```java
     List<List<Integer>> lists = Arrays.asList(Arrays.asList(1, 2), Arrays.asList(3, 4), Arrays.asList(5, 6));
     List<Integer> flatList = lists.stream()
                                  .flatMap(List::stream)
                                  .collect(Collectors.toList());
     System.out.println(flatList); // [1, 2, 3, 4, 5, 6]
     ```

4. `distinct()`

   - 功能：去重，返回一个新的流，包含不重复的元素

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 4);
     List<Integer> distinctNumbers = numbers.stream()
                                            .distinct()
                                            .collect(Collectors.toList());
     System.out.println(distinctNumbers); // [1, 2, 3, 4]
     ```

5. `sorted()`

   - 功能：对流中的元素进行排序，返回一个新的排序后的流

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 2);
     List<Integer> sortedNumbers = numbers.stream()
                                          .sorted()
                                          .collect(Collectors.toList());
     System.out.println(sortedNumbers); // [1, 2, 3, 5, 8]
     ```

   - 如果不传入任何 `Comparator` ，默认是升序排序。想要降序排序的话，可以传入 Java 提供的 `Comparator.reverseOrder()` 来简化降序排序。

     ```java
     List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 3);
     
     // 使用 sorted() 和 Comparator.reverseOrder() 进行降序排序
     List<Integer> sortedNumbersDesc = numbers.stream()
                                         .sorted(Comparator.reverseOrder())
                                         .collect(Collectors.toList());
     
     System.out.println(sortedNumbersDesc); // 输出：[8, 5, 3, 2, 1]
     ```

6. `peek()`

   - 功能：对流中的每个元素进行操作，**通常用于调试**（例如输出元素），并返回一个新的流。**注意**：`peek()` 是一个中间操作，它不会修改流中的元素，而是返回原始流

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
     numbers.stream()
            .peek(n -> System.out.println("Processing: " + n))
            .map(n -> n * 2)
            .collect(Collectors.toList());
     // 输出：
     // Processing: 1
     // Processing: 2
     // Processing: 3
     // Processing: 4
     // Processing: 5
     ```

7. `limit()`

   - 功能：返回流中前 n 个元素，常用于分页或只处理前几个元素

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
     List<Integer> limitedNumbers = numbers.stream()
                                           .limit(3)
                                           .collect(Collectors.toList());
     System.out.println(limitedNumbers); // [1, 2, 3]
     ```

8. `skip()`

   - 功能：跳过流中的前 n 个元素，返回剩余的元素

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
     List<Integer> skippedNumbers = numbers.stream()
                                           .skip(2)
                                           .collect(Collectors.toList());
     System.out.println(skippedNumbers); // [3, 4, 5]
     ```



### 常用的终结操作

1. `collect()`

   - 功能：将流中的元素收集到集合或其他形式的容器中，最常用来将流转换为 `List`、`Set` 或 `Map`

   - 示例：

     ```java
     List<String> words = Arrays.asList("apple", "banana", "cherry");
     Set<String> wordSet = words.stream()
                                .collect(Collectors.toSet());
     System.out.println(wordSet); // [apple, banana, cherry]
     ```

2. `forEach()`

   - 功能：对流中的每个元素执行一个操作，常用于打印或遍历

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
     numbers.stream().forEach(System.out::println); // 打印每个元素
     ```

3. `reduce()`

   - 功能：通过累积操作将流中的元素合并为一个结果，常用于求和、求最大值、求最小值等操作

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
     int sum = numbers.stream()
                      .reduce(0, (a, b) -> a + b); // 求和
     System.out.println(sum); // 15
     ```

4. `count()`

   - 功能：返回流中元素的数量

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
     long count = numbers.stream()
                         .count();
     System.out.println(count); // 5
     ```

5. `anyMatch()` / `allMatch()` / `noneMatch()`

   - 功能：用于检查流中的元素是否满足某个条件

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
     boolean hasEven = numbers.stream().anyMatch(n -> n % 2 == 0);
     System.out.println(hasEven); // true
     ```

6. `findFirst()` / `findAny()`

   - 功能：返回流中的第一个元素（`findFirst()`）或任意一个元素（`findAny()`）

   - 示例：

     ```java
     List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
     Optional<Integer> first = numbers.stream().findFirst();
     System.out.println(first.orElse(-1)); // 1
     ```



