---
layout: post
title: python + django + drf
description: 记录
tag: 笔记
---

## python

**文件类型相关**

python 是动态类型语言，存在三种文件类型：python file、python unit test 和 python stub

- python file：普通的 Python 脚本文件，用于编写常规的 Python 代码，如业务逻辑、工具脚本、程序主体等
- python unit test：专门用于编写单元测试的文件，通常与测试框架（如 `unittest`、`pytest`）配合使用，用于验证代码功能的正确性
- python stub：存根文件（`.pyi`），用于类型提示（Type Hints），不包含实际代码逻辑，仅定义函数、类、变量的类型签名

最后一个 python stub可能不太好理解，由于 python 是动态类型语言，写代码时不强制声明变量类型，但在大型项目里，不明确类型会让代码阅读和维护变难，IDE 也不好做智能提示，Python stub 就是专门用来写类型信息的文件，它里面只写函数、变量、类等的类型，没有实际代码逻辑



**变量**

python 中对象分为可变对象和不可变对象，主要区别体现在 **对象的值能否被修改**

- **不可变对象**：一旦创建，其值就不能被修改。**若要修改值，实际上是创建了一个新对象，原对象保持不变**。不可变对象有：int、float、str、tuple 和 bool 等等
- **可变对象**：对象创建后，其值可以在原地被修改，不会创建新的对象。主要有：list、dict 和 set 等

```python
cnt = 2
print(id(cnt))
cnt += 2
print(id(cnt))
```

比如上面代码，cnt 前后指向的是不同的对象，输出结果如下：

```
140711354237896
140711354237960
```

**python 中没有内置的常量类型**，但通常约定使用 **全大写** 来指出应将某个变量视为常量

```python
MAX_CONNECTIONS = 5000
```



**列表**

python 中的 `list` 本质上是一个动态数组，也就是可变长度的数组

- **动态数组**：在内存中连续分配，通过索引能够实现随机访问，时间复杂度为 `O(1)`
- **空间分配策略**：为避免频繁的内存分配，`list` 在创建时会 **预先分配比实际所需更多的内存空间**。当元素增多并超出已分配的空间时，`list` 会重新分配更大的内存空间，然后把原有元素复制到新空间，释放旧内存
- 补充：`tuple` 和 `list`
  - `tuple` 是不可变序列，底层实现为静态数组，元组的元素在内存中也是连续分配的
  - 由于 `tuple` 不可变，在创建时就会根据元素数量一次性分配 **固定大小** 的内存空间

删除 `list` 中的元素时，**如果知道要删除元素的索引位置**，可以使用 `pop()` 或者 `del`

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
# 删除 'yamaha'
del motorcycles[1]
print(motorcycles)
# 默认弹出末尾元素，可以指定索引，此处删除的是 'suzuki'
print(motorcycles.pop())
print(motorcycles)
```

但区别是 `del` 无法拿到删除的元素，`pop()` 可以拿到删除的元素，输出结果如下：

```
['honda', 'suzuki']
suzuki
['honda']
```

如果说我们不知道元素的索引位置，但知道我们想删除的内容，可以使用 `remove()`

```python
motorcycles = ['honda', 'yamaha', 'suzuki', 'ducati']
print(motorcycles)
motorcycles.remove('ducati')
print(motorcycles)
```

输出结果：

```
['honda', 'yamaha', 'suzuki', 'ducati']
['honda', 'yamaha', 'suzuki']
```

`remove()` 和 `pop()` 类似，也可以返回被删除的元素，另外，**方法 `remove()` 只删除第一个指定的值**。如果要删除的值可能在列表中 出现多次，就需要使用循环来确保将每个值都删除

