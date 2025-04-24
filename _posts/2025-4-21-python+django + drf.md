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

对于列表复制，需要使用切片，如果直接赋值，两个列表指向的会是同一个对象

```python
players = ['charles', 'martina', 'michael', 'florence', 'eli']
# 同一个对象
players2 = players
# 副本，不同的对象
players3 = players[:]
print(id(players))
print(id(players2))
print(id(players3))
```

输出结果：

```
2011801227200
2011801227200
2011800984832
```

**面向对象**

以下面的代码为例，了解 python 中的面向对象思想

```python
class Dog:
    # 类属性
    species = "Canis familiaris"
    
    # 在 __init__ 中添加实例属性
    def __init__(self, name, age):
        self.name = name
        self.age = age

    # 动态添加实例属性
    def set_age(self, age):
        self.age = age
        
    def sit(self):
        print(f"{self.name} is now sitting.")

    def roll_over(self):
        print(f"{self.name} rolled over.")


my_dog = Dog('Willie', 6)
print(f"My dog's name is {my_dog.name}")
print(f"My dog is {my_dog.age} years old")
my_dog.sit()
my_dog.roll_over()
```

python 是动态类型语言，定义类时不需要像 java 那样单独把类中的属性显示写出来

在类中定义属性有两种常见方式：**类属性** 和 **实例属性**

- **类属性**：属于类本身的属性，**所有实例共享该属性**，可以在类的定义中直接赋值来定义类属性，比如上面代码的 `species`

- **实例属性**：属于类的每个实例，通常在 `__init__` 方法中通过 `self` 参数来定义和初始化实例属性，也可以在其他实例方法中动态添加

在 python 中，不存在像 java 那种严格意义上的私有属性（使用 `private` 修饰），但可以通过 **单下划线前缀（弱私有约定）** 来限制访问

```python
class Person:
    def __init__(self, name, age):
        self._name = name
        self._age = age

    def _display_info(self):
        print(f"Name: {self._name}, Age: {self._age}")

p = Person("Alice", 25)
# 可以访问，但不建议
print(p._name)  
p._display_info()
```

在上述代码中，`_name`、`_age` 和 `_display_info` 以单下划线开头，按照约定应在类内部使用，**但仍然可以在类外部访问它们** 。因此这仅仅是一种约定，而非强制性的修饰符

**类的继承**

```python
# 父类
class Car:
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0

    def get_descriptive_name(self):
        long_name = f"{self.year} {self.make} {self.model}"
        return long_name.title()

    def read_odometer(self):
        print(f"This car has {self.odometer_reading} miles on it.")

    def update_odometer(self, mileage):
        if mileage >= self.odometer_reading:
            self.odometer_reading += mileage
        else:
            print("You can't roll back odometer!")
            
    def increment_odometer(self, miles):
        self.odometer_reading += miles


# 子类
class ElectricCar(Car):
    def __init__(self, make, model, year):
        super().__init__(make, model, year)


my_tesla = ElectricCar('tesla', 'models', 2019)
```

创建子类时，父类 **必须包含** 在当前文件中，且位于子类前面。这里的 **必须包含** 不是说父类与子类必须写在同一个 `.py` 文件中，我们也可以把父类和子类分别写在不同的 `.py` 文件里，比如下面的代码：

```python
# animal.py
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        print(f"{self.name} makes a sound.")
```

```python
# dog.py
from animal import Animal

class Dog(Animal):
    def speak(self):
        print(f"{self.name} barks.")

# 创建子类实例并调用方法
dog = Dog("Buddy")
dog.speak()
```

运行 `dog.py` 时，python 会先从 `animal.py` 文件里导入 `Animal` 类，然后创建 `Dog` 类的实例并调用其方法

**重写父类方法**

父类方法不符合子类模拟的实物行为时，都可以进行重写。为此，可在子类中定义一个与要重写的父类方法同名的方法，这样，python 将不考虑父类方法，只关注子类方法

