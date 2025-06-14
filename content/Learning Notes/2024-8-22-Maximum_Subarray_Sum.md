---
layout: post
title: 最大子数组和
description: 记录
tag: 算法
---

对于最大子数组和的动态规划问题，一般这样思考：

定义状态 `f[i]` 表示以 `a[i]` 结尾的最大子数组和，不和 `i` 左边拼起来就是 `f[i] = a[i]` ，和 `i` 左边拼起来就是 `f[i] = f[i-1] + a[i]` ，取最大值就得到了状态转移方程 `f[i] = max(f[i-1], 0) + a[i]` ，答案为 `max(f)` 。这种做法也称为 Kadane 算法。



## 1.最大子数组和

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。



**子数组**

是数组中的一个连续部分。



 

**示例 1：**

```
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```

**示例 2：**

```
输入：nums = [1]
输出：1
```

**示例 3：**

```
输入：nums = [5,4,-1,7,8]
输出：23
```

 

**提示：**

- `1 <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`

 

### 思路

按照前面说的进行状态转移即可



### Code

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp(n, 0);
        dp[0] = nums[0];
        for (int i = 1; i < n; ++i) {
            dp[i] = max(dp[i-1], 0) + nums[i];
        }
        return ranges::max(dp);
    }
};
```





## 2.找到最大开销的子字符串

给你一个字符串 `s` ，一个字符 **互不相同** 的字符串 `chars` 和一个长度与 `chars` 相同的整数数组 `vals` 。

**子字符串的开销** 是一个子字符串中所有字符对应价值之和。空字符串的开销是 `0` 。

**字符的价值** 定义如下：

- 如果字符不在字符串 

  ```
  chars
  ```

   中，那么它的价值是它在字母表中的位置（下标从

   

  1

   开始）。

  - 比方说，`'a'` 的价值为 `1` ，`'b'` 的价值为 `2` ，以此类推，`'z'` 的价值为 `26` 。

- 否则，如果这个字符在 `chars` 中的位置为 `i` ，那么它的价值就是 `vals[i]` 。

请你返回字符串 `s` 的所有子字符串中的最大开销。

 

**示例 1：**

```
输入：s = "adaa", chars = "d", vals = [-1000]
输出：2
解释：字符 "a" 和 "d" 的价值分别为 1 和 -1000 。
最大开销子字符串是 "aa" ，它的开销为 1 + 1 = 2 。
2 是最大开销。
```

**示例 2：**

```
输入：s = "abc", chars = "abc", vals = [-1,-1,-1]
输出：0
解释：字符 "a" ，"b" 和 "c" 的价值分别为 -1 ，-1 和 -1 。
最大开销子字符串是 "" ，它的开销为 0 。
0 是最大开销。
```

 

**提示：**

- `1 <= s.length <= 10^5`
- `s` 只包含小写英文字母。
- `1 <= chars.length <= 26`
- `chars` 只包含小写英文字母，且 **互不相同** 。
- `vals.length == chars.length`
- `-1000 <= vals[i] <= 1000`



### 思路

和 T1 比，多了个计算字符串开销的操作，大体思路几乎一样



### Code

```c++
class Solution {
public:
    int maximumCostSubstring(string s, string chars, vector<int>& vals) {
        int len = s.length(), res = 0;
        vector<int> dp(len, 0);
        auto first = chars.find(s[0]);
        if (first != string::npos) {
            dp[0] = max(0, vals[first]);
        } else {
            dp[0] = s[0] - 'a' + 1;
        }
        for (int i = 1; i < len; ++i) {
            auto index = chars.find(s[i]);
            if (index != string::npos) {
                dp[i] = max(dp[i-1], 0) + vals[index];
            } else {
                dp[i] = max(dp[i-1], 0) + s[i] - 'a' + 1;
            }
        }
        return ranges::max(dp);
    }
};
```







## 3.任意子数组和的绝对值的最大值

给你一个整数数组 `nums` 。一个子数组 `[numsl, numsl+1, ..., numsr-1, numsr]` 的 **和的绝对值** 为 `abs(numsl + numsl+1 + ... + numsr-1 + numsr)` 。

请你找出 `nums` 中 **和的绝对值** 最大的任意子数组（**可能为空**），并返回该 **最大值** 。

`abs(x)` 定义如下：

- 如果 `x` 是负整数，那么 `abs(x) = -x` 。
- 如果 `x` 是非负整数，那么 `abs(x) = x` 。

 

**示例 1：**

```
输入：nums = [1,-3,2,3,-4]
输出：5
解释：子数组 [2,3] 和的绝对值最大，为 abs(2+3) = abs(5) = 5 。
```

**示例 2：**

```
输入：nums = [2,-5,1,-4,3,-2]
输出：8
解释：子数组 [-5,1,-4] 和的绝对值最大，为 abs(-5+1-4) = abs(-8) = 8 。
```

 

**提示：**

- `1 <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`



### 思路

要求数组中**任意子数组和的绝对值**的最大值，要么正值尽可能大，要么负值尽可能小，所以我们分别计算最大子数组和 `f_max` 与最小子数组和 `f_min` ，取它们绝对值的较大值即可



### Code

```c++
class Solution {
public:
    int maxAbsoluteSum(vector<int>& nums) {
        int res = 0, f_max = 0, f_min = 0;
        for (auto x: nums) {
            f_max = max(f_max, 0) + x;
            f_min = min(f_min, 0) + x;
            res = max({res, f_max, -f_min});
        }
        return res;
    }
};
```







## 4. K 次串联后最大子数组之和

给定一个整数数组 `arr` 和一个整数 `k` ，通过重复 `k` 次来修改数组。

例如，如果 `arr = [1, 2]` ， `k = 3` ，那么修改后的数组将是 `[1, 2, 1, 2, 1, 2]` 。

返回修改后的数组中的最大的子数组之和。注意，子数组长度可以是 `0`，在这种情况下它的总和也是 `0`。

由于 **结果可能会很大**，需要返回的 `109 + 7` 的 **模** 。

 

**示例 1：**

```
输入：arr = [1,2], k = 3
输出：9
```

**示例 2：**

```
输入：arr = [1,-2,1], k = 5
输出：2
```

**示例 3：**

```
输入：arr = [-1,-2], k = 7
输出：0
```

 

**提示：**

- `1 <= arr.length <= 10^5`
- `1 <= k <= 10^5`
- `-10^4 <= arr[i] <= 10^4`



### 思路

首先，不能直接拼接 k 次给定的数组，数据范围是 `k * length = 10^10` ，会爆内存。

我们分下面两种情况考虑：

- `k == 1` 时，就是求最大子数组和，直接 dp 即可
- `k >= 2` 时，**因为数组 `arr` 是不断重复的**，我们要考虑的是：“**最大子数组和是出现在一个 `arr` 中还是在两个 `arr` 的交界处**”， 所以我们只需要考虑整个重复数组的头尾两个数组**（理解成给定的 `arr` 重复 `k` 次，只考虑第 `1` 次出现的 `arr` 和第 `k` 次出现的 `arr` ，这两个 `arr` 之间还有 `k-2` 个 `arr` ，如果理解不了就多想几次）**，之后我们计算单个 `arr` 的所有元素之和 `sum` ，对于 `sum` 又有下面两种情况：
  - `sum >= 0` 时，我们就可以在第 `1` 个 `arr` 和第  `k` 个 `arr` 之间插入 `k-2` 个 `sum` 来构成更大的子数组和（因为 `k` 次串联后的数组是连续的）
  - `sum < 0` 时，我们就直接不要插入 `sum` ，直接返回即可



### Code

```c++
class Solution {
public:
    int kConcatenationMaxSum(vector<int>& arr, int k) {
        int ans = 0, f = 0;
        if (k == 1) {
            for (auto x: arr) {
                f = max(f, 0) + x;
                ans = max(ans, f);
            }
        } else {
            int n = arr.size();
            for (int i = 0; i < 2 * n; ++i) {
                f = max(f, 0) + arr[i%n];
                ans = max(ans, f);
            }
            int sum = 0, mod = 1e9 + 7;
            for (auto x: arr) sum += x;
            if (sum > 0) ans = (ans + (k - 2ll) * sum) % mod;
        }
        return ans;
    }
};
```





## 5.环形子数组的最大和

给定一个长度为 `n` 的**环形整数数组** `nums` ，返回 *`nums` 的非空 **子数组** 的最大可能和* 。

**环形数组** 意味着数组的末端将会与开头相连呈环状。形式上， `nums[i]` 的下一个元素是 `nums[(i + 1) % n]` ， `nums[i]` 的前一个元素是 `nums[(i - 1 + n) % n]` 。

**子数组** 最多只能包含固定缓冲区 `nums` 中的每个元素一次。形式上，对于子数组 `nums[i], nums[i + 1], ..., nums[j]` ，不存在 `i <= k1, k2 <= j` 其中 `k1 % n == k2 % n` 。

 

**示例 1：**

```
输入：nums = [1,-2,3,-2]
输出：3
解释：从子数组 [3] 得到最大和 3
```

**示例 2：**

```
输入：nums = [5,-3,5]
输出：10
解释：从子数组 [5,5] 得到最大和 5 + 5 = 10
```

**示例 3：**

```
输入：nums = [3,-2,2,-3]
输出：3
解释：从子数组 [3] 和 [3,-2,2] 都可以得到最大和 3
```

 

**提示：**

- `n == nums.length`
- `1 <= n <= 3 * 10^4`
- `-3 * 10^4 <= nums[i] <= 3 * 10^4`



### 思路

![lc918-c.png](https://pic.leetcode.cn/1689750394-drKSAI-lc918-c.png)



### Code

```c++
class Solution {
public:
    int maxSubarraySumCircular(vector<int>& nums) {
        int max_s = INT_MIN; // 最大子数组和
        int min_s = 0; // 最小子数组和
        int max_f = 0, min_f = 0, sum = 0;
        for (auto x: nums) {
            max_f = max(max_f, 0) + x;
            max_s = max(max_s, max_f);
            min_f = min(min_f, 0) + x;
            min_s = min(min_s, min_f);
            sum += x;
        }
        // sum - min_s 表示跨界的最大子数组和
        return sum == min_s ? max_s : max(max_s, sum - min_s);
    }
};
```







## 6.拼接数组的最大分数

给你两个下标从 **0** 开始的整数数组 `nums1` 和 `nums2` ，长度都是 `n` 。

你可以选择两个整数 `left` 和 `right` ，其中 `0 <= left <= right < n` ，接着 **交换** 两个子数组 `nums1[left...right]` 和 `nums2[left...right]` 。

- 例如，设 `nums1 = [1,2,3,4,5]` 和 `nums2 = [11,12,13,14,15]` ，整数选择 `left = 1` 和 `right = 2`，那么 `nums1` 会变为 `[1,***12\*,\*13\***,4,5]` 而 `nums2` 会变为 `[11,***2,3***,14,15]` 。

你可以选择执行上述操作 **一次** 或不执行任何操作。

数组的 **分数** 取 `sum(nums1)` 和 `sum(nums2)` 中的最大值，其中 `sum(arr)` 是数组 `arr` 中所有元素之和。

返回 **可能的最大分数** 。

**子数组** 是数组中连续的一个元素序列。`arr[left...right]` 表示子数组包含 `nums` 中下标 `left` 和 `right` 之间的元素**（含** 下标 `left` 和 `right` 对应元素**）**。

 

**示例 1：**

```
输入：nums1 = [60,60,60], nums2 = [10,90,10]
输出：210
解释：选择 left = 1 和 right = 1 ，得到 nums1 = [60,90,60] 和 nums2 = [10,60,10] 。
分数为 max(sum(nums1), sum(nums2)) = max(210, 80) = 210 。
```

**示例 2：**

```
输入：nums1 = [20,40,20,70,30], nums2 = [50,20,50,40,20]
输出：220
解释：选择 left = 3 和 right = 4 ，得到 nums1 = [20,40,20,40,20] 和 nums2 = [50,20,50,70,30] 。
分数为 max(sum(nums1), sum(nums2)) = max(140, 220) = 220 。
```

**示例 3：**

```
输入：nums1 = [7,11,13], nums2 = [1,1,1]
输出：31
解释：选择不交换任何子数组。
分数为 max(sum(nums1), sum(nums2)) = max(31, 3) = 31 。
```

 

**提示：**

- `n == nums1.length == nums2.length`
- `1 <= n <= 10^5`
- `1 <= nums1[i], nums2[i] <= 10^4`



### 思路

设：
$$
s_1 = \Sigma nums_1[i]
$$
交换 `[left, right]` 范围内的元素后，对于 `nums1'`  有
$$
\Sigma nums_1'[i] = s_1 - (nums_1[left] + ... + nums_1[right]) + (nums_2[left] + ... + nums_2[right])
$$
合并相同下标，等号右侧变形为：
$$
s_1 + (nums_2[left] - nums_1[left]) + ... + (nums_2[right] - nums_1[right])
$$
  设：`diff[i] = nums2[i] - nums1[i]` ，上式变为：
$$
s_1 + diff[left] + ... + diff[right]
$$
为了最大化上式，我们需要最大化 `diff` 数组的**最大子数组和（子数组可为空）**

对于 `nums2` 也同理，求这两者的最大值即为答案。



### Code

```c++
class Solution {
    int solve(vector<int>& nums1, vector<int>& nums2) {
        int s1 = 0, maxSum = 0;
        for (int i = 0, s = 0; i < nums1.size(); ++i) {
            s1 += nums1[i];
            s = max(s + nums2[i] - nums1[i], 0);
            maxSum = max(maxSum, s);
        }
        return s1 + maxSum;
    }

public:
    int maximumsSplicedArray(vector<int>& nums1, vector<int>& nums2) {
        return max(solve(nums1, nums2), solve(nums2, nums1));
    }
};
```

