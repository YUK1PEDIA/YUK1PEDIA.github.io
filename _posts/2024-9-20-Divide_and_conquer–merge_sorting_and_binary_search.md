---
layout: post
title: 分治——归并排序与二分查找
description: 记录
tag: 算法
---

# 分治——归并排序与二分查找

## 交易逆序对的总数

在股票交易中，如果前一天的股价高于后一天的股价，则可以认为存在一个「交易逆序对」。请设计一个程序，输入一段时间内的股票交易记录 `record`，返回其中存在的「交易逆序对」总数。

**示例 1:**

```
输入：record = [9, 7, 5, 4, 6]
输出：8
解释：交易中的逆序对为 (9, 7), (9, 5), (9, 4), (9, 6), (7, 5), (7, 4), (7, 6), (5, 4)。
```

 

**限制：**

```
0 <= record.length <= 50000
```



**Code**

```c++
class Solution {
public:
    int mergeSort(vector<int>& record, vector<int>& tmp, int l, int r) {
        if (l >= r) return 0;
        int mid = (l + r) / 2;
        int inv_cnt = mergeSort(record, tmp, l, mid) + mergeSort(record, tmp, mid + 1, r);
        int i = l, j = mid + 1, pos = l;
        while (i <= mid && j <= r) {
            if (record[i] <= record[j]) {
                tmp[pos] = record[i];
                ++i;
                inv_cnt += (j - (mid + 1));
            } else {
                tmp[pos] = record[j];
                ++j;
            }
            ++pos;
        }
        for (int k = i; k <= mid; ++k) {
            // 右边已经没有比左边当前数更大的了，直接加上 (j - (mid + 1))
            tmp[pos++] = record[k];
            inv_cnt += (j - (mid + 1));
        }
        for (int k = j; k <= r; ++k) {
            tmp[pos++] = record[k];
        }
        copy(tmp.begin() + l, tmp.begin() + r + 1, record.begin() + l);
        return inv_cnt;
    }

    int reversePairs(vector<int>& record) {
        int n = record.size();
        vector<int> tmp(n);
        return mergeSort(record, tmp, 0, n - 1);
    }
};
```







## 二分查找（闭区间写法）

```c++
int lower_bound(vector<int> &nums, int target) {
    int left = 0, right = (int) nums.size() - 1; // 闭区间 [left, right]
    while (left <= right) { // 区间不为空
        // 循环不变量：
        // nums[left-1] < target
        // nums[right+1] >= target
        // mid 也可以写成：int mid = ((right - left) >> 1) + left;
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1; // 范围缩小到 [mid+1, right]
        } else {
            right = mid - 1; // 范围缩小到 [left, mid-1]
        }
    }
    return left;
}
```



### 统计目标成绩的出现次数

某班级考试成绩按非严格递增顺序记录于整数数组 `scores`，请返回目标成绩 `target` 的出现次数。

 

**示例 1：**

```
输入: scores = [2, 2, 3, 4, 4, 4, 5, 6, 6, 8], target = 4
输出: 3
```

**示例 2：**

```
输入: scores = [1, 2, 3, 5, 7, 9], target = 6
输出: 0
```

 

**提示：**

- `0 <= scores.length <= 10^5`
- `-10^9 <= scores[i] <= 10^9`
- `scores` 是一个非递减数组
- `-10^9 <= target <= 10^9`



**Code**

```c++
class Solution {
public:
    int lower_bound(vector<int>& nums, int target) {
        int n = nums.size();
        int left = 0, right = n - 1;
        while (left <= right) {
            int mid = ((right - left) >> 1) + left;
            if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return left;
    }

    int countTarget(vector<int>& scores, int target) {
        int start = lower_bound(scores, target);
        int end = lower_bound(scores, target + 1);
        return end - start;
    }
};
```

