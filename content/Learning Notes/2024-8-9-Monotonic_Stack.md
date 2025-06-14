---
layout: post
title: 单调栈
description: 记录
tag: 算法
---
## 什么是单调栈？

单调栈是一种特殊的栈，在栈的「先进后出」规则基础上，要求「从 **栈顶** 到 **栈底** 的元素是单调递增（或者单调递减）」。其中满足从栈顶到栈底的元素是单调递增的栈，叫做「单调递增栈」。满足从栈顶到栈底的元素是单调递减的栈，叫做「单调递减栈」。





## 单调栈的 push 和 pop

比如我们现在有一组数：`[10, 3, 7, 4, 12]` ，如何将这组数放到一个**单调递增栈**中？

我们从左到右遍历数组，根据以下条件判断进行何种操作：

- **如果栈空或当前遍历到的元素 `cur` 小于栈顶元素**，则入栈；
- **如果栈非空并且当前遍历到的元素 `cur` 大于栈顶元素**，则将栈中所有比 `cur` 小的元素出栈，再将 `cur` 入栈。

对于上述数组，通过单调栈有如下操作：

1. 对于 `10` ，当前栈空，直接入栈，此时栈顶到栈底：`[10]`；
2. 对于 `3` ，当前栈非空，且当前元素小于栈顶元素，直接入栈，此时栈顶到栈底：`[3, 10]`；
3. 对于 `7` ，当前栈非空，且当前元素大于栈顶元素，先将栈顶元素 `3` 出栈，再将 `7` 入栈，此时栈顶到栈底：`[7, 10]`；
4. 对于 `4` ，当前栈非空，且当前元素小于栈顶元素，直接入栈，此时栈顶到栈底：`[4, 7, 10]`；
5. 对于 `12` ，当前栈非空，且当前元素大于栈顶元素，先将 `4、7、10`  出栈，再将 `12` 入栈，此时栈顶到栈底：`[12]`；





## 单调栈能解决什么问题？

当我们遇到：**需要在 `O(n)` 的时间复杂度内求出数组中各个元素右侧或左侧第一个更大或更小的元素及其下标，然后一并得到其他信息**时，均可以尝试单调栈的做法





## 例题

## 87.每日温度

给定一个整数数组 `temperatures` ，表示每天的温度，返回一个数组 `answer` ，其中 `answer[i]` 是指对于第 `i` 天，下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

 

**示例 1:**

```
输入: temperatures = [73,74,75,71,69,72,76,73]
输出: [1,1,4,2,1,1,0,0]
```

**示例 2:**

```
输入: temperatures = [30,40,50,60]
输出: [1,1,1,0]
```

**示例 3:**

```
输入: temperatures = [30,60,90]
输出: [1,1,0]
```

 

**提示：**

- `1 <= temperatures.length <= 10^5`
- `30 <= temperatures[i] <= 100`



### 思路

数据范围是 `10^5` 使用 `O(n^2)` 的复杂度一定会超时，这里可以考虑借助一个单调栈解决。

我们先定义一个长度为 `temperatures.size()` 的数组 `ans` ，然后定义一个单调栈来存放 `temperatures` 中元素的下标，**从栈底到栈顶存放的下标所对应的温度是非递增的**。我们遍历一遍 `temeratures` 数组，进行如下处理：

- **当栈非空并且栈顶元素小于当前遍历到的元素 `a` **，那么 `a` 一定是栈顶元素右边第一个比它大的元素，于是弹出栈顶元素 `pre` ，更新 `ans` 数组中栈顶元素对应的结果为 `i - pre` 
- **如若当前遍历到的元素小于等于栈顶元素或者栈空**，将其入栈

代码如下：

```c++
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        int n = temperatures.size();
        vector<int> ans(n);
        stack<int> stk;
        for (int i = 0; i < n; ++i) {
            while (!stk.empty() && temperatures[i] > temperatures[stk.top()]) {
                int pre = stk.top();
                ans[pre] = i - pre;
                stk.pop();
            }
            stk.push(i);
        }
        return ans;
    }
};
```







## 89.柱状图中最大的矩形

给定 *n* 个非负整数，用来表示柱状图中各个柱子的高度。每个柱子彼此相邻，且宽度为 1 。

求在该柱状图中，能够勾勒出来的矩形的最大面积。

 

**示例 1:**

![img](https://assets.leetcode.com/uploads/2021/01/04/histogram.jpg)

```
输入：heights = [2,1,5,6,2,3]
输出：10
解释：最大的矩形为图中红色区域，面积为 10
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/01/04/histogram-1.jpg)

```
输入： heights = [2,4]
输出： 4
```

 

**提示：**

- `1 <= heights.length <=10^5`
- `0 <= heights[i] <= 10^4`



### 思路

首先，**面积最大的矩形高度一定是 `heights` 中的元素**。这可以用反证法证明：假如高度不在 `heights` 中，比如 `4` ，那我们可以增加高度直到触及某根柱子的顶部，比如增加到 `5` ，由于矩形底边长不变，高度增加，我们得到了面积更大的矩形，矛盾，所以面积最大矩形的高度一定是 `heights` 中的元素。

假设 `h = heights[i]` 是矩形的**高度**，那么矩形的**宽度**最大是多少？这是我们要求的数据。我们需要知道：

- 在 `i` 左侧的小于 `h` 的最近元素的下标为 `left` ，如果不存在则为 `-1` 。这样 `left + 1` 就是在 `i` 左侧的大于等于 `h` 的最近元素的下标。
- 在 `i` 右侧的小于 `h` 的最近元素的下标为 `right` ，如果不存在则为 `n` 。这样 `right - 1` 就是在 `i` 右侧的大于等于 `h` 的最近元素的下标。

比如示例 1 ，选择 `i = 2` 这个柱子作为矩形高，那么左边小于 `heights[2] = 5` 的最近元素的下标为 `left = 1` ，右边小于 `heights[2] = 5` 的最近元素的下标为 `right = 4` 。

**那么矩形的宽度就是 `right - left - 1 = 4 - 1 - 1 = 2` ，矩形面积为 `h * (right - left - 1) = 5 * 2 = 10` 。**

通过枚举 `heights[i]` ，计算每个 `heights[i]` 对应的矩形面积，更新答案的最大值。快速计算 `left` 和 `right` 可以用单调栈实现

**总的思路：求最大面积 -> 枚举每个 `heights[i]` -> 找到每个 `heights[i]` 对应的最大宽度 -> 得到每个 `heights[i]` 对应的最大面积 -> 再枚举每个最大面积，找到最终结果**

代码如下：

```c++
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int n = heights.size();
        // 记录每个 heights[i] 左侧的最近元素下标
        vector<int> left(n, -1);
        stack<int> st; // 利用单调栈计算 left 和 right，原理可以看 T87：每日温度
        for (int i = 0; i < n; ++i) {
            // 利用单调栈计算每个 heights[i] 左侧的最近元素下标
            while (!st.empty() && heights[i] <= heights[st.top()]) {
                st.pop();
            }
            if (!st.empty()) {
                left[i] = st.top();
            }
            st.push(i);
        }

        // 每个 heights[i] 右侧的最近元素下标也同样用单调栈计算
        vector<int> right(n, n);
        st = stack<int>();
        for (int i = n - 1; i >= 0; --i) {
            while (!st.empty() && heights[i] <= heights[st.top()]) {
                st.pop();
            }
            if (!st.empty()) {
                right[i] = st.top();
            }
            st.push(i);
        }

        int ans = 0; 
        // 更新每个高度对应的面积
        for (int i = 0; i < n; ++i) {
            ans = max(ans, heights[i] * (right[i] - left[i] - 1));
        }
        return ans;
    }
};
```

