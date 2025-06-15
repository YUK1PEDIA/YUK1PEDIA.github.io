---
layout: post
title: LCR-栈和队列
description: 记录
tag: 算法
---

## 1.用两个栈实现队列

读者来到图书馆排队借还书，图书管理员使用两个书车来完成整理借还书的任务。书车中的书从下往上叠加存放，图书管理员每次只能拿取书车顶部的书。排队的读者会有两种操作：

- `push(bookID)`：把借阅的书籍还到图书馆。
- `pop()`：从图书馆中借出书籍。

为了保持图书的顺序，图书管理员每次取出供读者借阅的书籍是 **最早** 归还到图书馆的书籍。你需要返回 **每次读者借出书的值** 。

如果没有归还的书可以取出，返回 `-1` 。

 

**示例 1：**

```
输入：
["BookQueue", "push", "push", "pop"]
[[], [1], [2], []]
输出：[null,null,null,1]
解释：
MyQueue myQueue = new MyQueue();
myQueue.push(1); // queue is: [1]
myQueue.push(2); // queue is: [1, 2] (leftmost is front of the queue)
myQueue.pop(); // return 1, queue is [2]
```

 

**提示：**

- `1 <= bookID <= 10000`
- 最多会对 `push`、`pop` 进行 `10000` 次调用



**思路**

用两个栈 *st1* 和 *st2* 实现队列，我们将 *st2* 作为辅助栈，其栈顶元素即为队首元素。

调用删除队首元素函数时，若 *st2* 不为空，那么直接返回其栈顶元素，否则先将 *st1* 通过 *st2* 反转，再返回 *st2* 的栈顶元素。



**Code**

```c++
class CQueue {
private:
    stack<int> st1, st2;

public:
    CQueue() {
    }
    
    void appendTail(int value) {
        st1.push(value);
    }
    
    int deleteHead() {
        int res = 0;
        if (!st2.empty()) {
            res = st2.top();
            st2.pop();
        } else {
            if (st1.empty()) return -1;
            while (!st1.empty()) {
                st2.push(st1.top());
                st1.pop();
            }
            res = st2.top();
            st2.pop();
        }
        return res;
    }
};

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue* obj = new CQueue();
 * obj->appendTail(value);
 * int param_2 = obj->deleteHead();
 */
```





## 2.最小栈

请你设计一个 **最小栈** 。它提供 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

 

实现 `MinStack` 类:

- `MinStack()` 初始化堆栈对象。
- `void push(int val)` 将元素val推入堆栈。
- `void pop()` 删除堆栈顶部的元素。
- `int top()` 获取堆栈顶部的元素。
- `int getMin()` 获取堆栈中的最小元素。

 

**示例 1:**

```
输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[2],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,2,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(2);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 2.
minStack.getMin();   --> 返回 -2.
```

 

 **
提示：**

- `-2^31 <= val <= 2^31 - 1`
- `pop`、`top` 和 `getMin` 操作总是在 **非空栈** 上调用
- `push`、`pop`、`top` 和 `getMin` 最多被调用 `3 * 10^4` 次





**思路**

使用两个栈，其中一个作为辅助栈用来维护栈中元素的最小值。

对于 *push* 操作，由于辅助栈 *st2* 的栈顶元素是栈中元素的最小值，所以需要判断入栈元素与 *st2* 栈顶元素的大小关系：

- 如果栈顶元素**大于或等于**入栈元素，那么将入栈元素加入辅助栈

对于 *pop* 操作，需要判断辅助栈 *st2* 栈顶元素和普通栈 *st1* 栈顶元素是否相等：

- 如果相等，辅助栈也要 *pop* 



**Code**

```c++
class MinStack {
private:
    stack<int> st1, st2;

public:
    /** initialize your data structure here. */
    MinStack() {
    }
    
    void push(int x) {
        st1.push(x);
        if (st2.empty() || st2.top() >= x) {
            st2.push(x);
        }
    }
    
    void pop() {
        if (st1.top() == st2.top()) {
            st2.pop();
        }
        st1.pop();
    }
    
    int top() {
        return st1.top();
    }
    
    int getMin() {
        return st2.top();
    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack* obj = new MinStack();
 * obj->push(x);
 * obj->pop();
 * int param_3 = obj->top();
 * int param_4 = obj->getMin();
 */
```





## 3.验证给定序列是否满足栈的压入和弹出顺序

现在图书馆有一堆图书需要放入书架，并且图书馆的书架是一种特殊的数据结构，只能按照 **一定** 的顺序 **放入** 和 **拿取** 书籍。

给定一个表示图书放入顺序的整数序列 `putIn`，请判断序列 `takeOut` 是否为按照正确的顺序拿取书籍的操作序列。你可以假设放入书架的所有书籍编号都不相同。

 

**示例 1：**

```
输入：putIn = [6,7,8,9,10,11], takeOut = [9,11,10,8,7,6]
输出：true
解释：我们可以按以下操作放入并拿取书籍：
push(6), push(7), push(8), push(9), pop() -> 9,
push(10), push(11),pop() -> 11,pop() -> 10, pop() -> 8, pop() -> 7, pop() -> 6
```

**示例 2：**

```
输入：putIn = [6,7,8,9,10,11], takeOut = [11,9,8,10,6,7]
输出：false
解释：6 不能在 7 之前取出。
```

 

**提示：**

- `0 <= putIn.length == takeOut.length <= 1000`
- `0 <= putIn[i], takeOut < 1000`
- `putIn` 是 `takeOut` 的排列。



**思路**

直接模拟入栈和出栈操作：

- 遍历 *putIn* 数组，不断将元素入栈
- 用一个指针 *j* 记录 *takeOut* 的当前出栈元素

如果 *takeOut* 的当前出栈元素和栈顶元素相等，那么将栈顶元素出栈，*j* 后移，遍历完成后判断栈是否为空：

- 如果栈为空，那么两个序列合理，否则不合理



**Code**

```c++
class Solution {
public:
    bool validateBookSequences(vector<int>& putIn, vector<int>& takeOut) {
        stack<int> st;
        int j = 0;
        for (auto& x: putIn) {
            st.push(x);
            while (!st.empty() && takeOut[j] == st.top()) {
                st.pop();
                ++j;
            }
        }
        return st.empty();
    }
};
```







## 4.队列的最大值

请设计一个自助结账系统，该系统需要通过一个队列来模拟顾客通过购物车的结算过程，需要实现的功能有：

- `get_max()`：获取结算商品中的最高价格，如果队列为空，则返回 -1
- `add(value)`：将价格为 `value` 的商品加入待结算商品队列的尾部
- `remove()`：移除第一个待结算的商品价格，如果队列为空，则返回 -1

注意，为保证该系统运转高效性，以上函数的均摊时间复杂度均为 O(1)

 

**示例 1：**

```
输入: 
["Checkout","add","add","get_max","remove","get_max"]
[[],[4],[7],[],[],[]]

输出: [null,null,null,7,4,7]
```

**示例 2：**

```
输入: 
["Checkout","remove","get_max"]
[[],[],[]]

输出: [null,-1,-1]
```

 

**提示：**

- `1 <= get_max, add, remove 的总操作数 <= 10000`
- `1 <= value <= 10^5`



**思路**

我们要**实时**获得队列中的最大值，可以用一个双向队列实现。这个双向队列维护一个**非严格递减序列**，队首元素对应当前队列中的最大值。

那么我们如何构造这样的一个双向队列 *deq* 呢？我们可以按照下面的方法构造

- 当 *deq* 为空，那么直接将当前入队元素 *value* 加入 *deq* 中；
- 如果不为空，不停弹出 *deq* 中小于 *value* 的队尾元素，然后将 *value* 加入队尾

同样的，对于出队操作，仍然需要判断 *deq* 中的队首元素和 *q* 的队首元素是否相等



**Code**

```c++
class Checkout {
private:
    queue<int> q;
    deque<int> deq;

public:
    Checkout() {
    }
    
    int get_max() {
        return q.empty() ? -1 : deq.front();
    }
    
    void add(int value) {
        q.push(value);
        while (!deq.empty() && value > deq.back()) {
            deq.pop_back();
        }
        deq.push_back(value);
    }
    
    int remove() {
        if (q.empty()) return -1;
        int res = q.front();
        if (deq.front() == res) deq.pop_front();
        q.pop();
        return res;
    }
};

/**
 * Your Checkout object will be instantiated and called as such:
 * Checkout* obj = new Checkout();
 * int param_1 = obj->get_max();
 * obj->add(value);
 * int param_3 = obj->remove();
 */
```

