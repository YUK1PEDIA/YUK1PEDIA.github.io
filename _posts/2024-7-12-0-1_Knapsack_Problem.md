---
layout: post
title: 01背包
description: 记录
tag: 算法
---

## 1.概念

有 `n` 件物品和一个最多能装重量为 `w` 的背包，第 `i` 件物品的重量是 `weight[i]`， 该物品的价值是 `value[i]` ，**每件物品只能用一次**，求解将哪些物品装入背包里能得到最大价值。

## 2.二维求解纯01背包问题

- 确定 `dp` 数组以及下标的含义
  - 对于背包问题有一种二维数组的写法：`dp[i][j]` 表示从下标为 `[0-i]` 的物品里任意取，放进容量为 `j` 的背包，价值总和最大是多少
- 确定状态转移方程
  - 我们需要根据 `dp` 数组以及下标的含义，**将原问题抽象成可以求解的子问题**，在这个过程中，我们要始终记着 `dp` 数组的含义。
  - 对于纯01背包问题，我们可以从遍历单个物品的角度去思考，这样也比较好理解：
    - **不放物品 i** ：`dp[i][j]` 可以由 `dp[i-1][j]` 推出，根据前面定义的 `dp` 数组，我们可以得知 `dp[i-1][j]` 表示背包容量为 `j` ，里面不放物品 `i` 的最大价值总和
    - **放物品 i** ：`dp[i][j]` 可以由 `dp[i-1][j-weight[i]]` 推出。可以类比着不放物品 `i` 的情况理解。那么此时 `dp[i][j] = dp[i-1][j-weight[i]] + value[i]` 
    - 遍历物品时对以上两种情况取一个 `max` 即可。

- `dp` 数组的初始化
  - 根据定义的 `dp` 数组的含义，我们对边界情况进行初始化
  - 从定义出发，如果容量 `j` 为 0 ，即 `dp[i][0]` ，那么肯定选不了任何物品，所以价值总和一定为 0 
  - 再从状态转移方程出发：
    - `dp[i][j] = max(dp[i-1][j], dp[i-1][j-weight[i]] + value[i])`
    - 防止数组越界，当 `i = 0` 时一定要初始化。在这个维度上，我们就要考虑背包容量了，背包容量必须大于等于第 0 个物品的重量才能装进去，其余情况均不能装，价值为 0
- 确定遍历顺序
  - **其实先遍历物品还是先遍历背包都可以**，但是先遍历物品更好理解，所以平常解题时都习惯于先遍历物品
  - 这里主要是要考虑**背包容量大小的遍历顺序**，遍历物品的顺序是固定的，从头往后遍历物品即可，**但是对于背包容量，从大往小遍历和从小往大遍历得到的结果是不同的**
- 01背包模板代码

```c++
int test_2_wei_bag_problem() {
    vector<int> weight = {1, 3, 4};
    vector<int> value = {15, 20, 30};
    int bagweight = 4;

    vector<vector<int>> dp(weight.size(), vector<int>(bagweight + 1, 0));

    for (int j = weight[0]; j <= bagweight; j++) {
        dp[0][j] = value[0];
    }

    for (int i = 1; i < weight.size(); i++) {
        for (int j = 0; j <= bagweight; j++) {
            if (j < weight[i]) {
                dp[i][j] = dp[i - 1][j];
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
            }
        }
    }
    return dp[weight.size() - 1][bagweight];
}
```



## 3.一维求解纯01背包问题

- 对于纯01背包问题，我们可以用一维数组实现，这时候 `dp[j]` 的含义就是容量为 `j` 的背包所能装的最大价值为 `dp[j]` 

-  状态转移方程与二维类似

  - `dp[j] = max(dp[j], dp[j-weight[i]] + value[i])`

- 初始化：只用考虑背包容量为 0 时，`dp[0] = 0` 

- 至于一维数组的遍历顺序，需要注意**遍历背包容量需要从大往小遍历**

  - 因为从大往小遍历（倒序遍历）可以保证 `物品 i` 只被放入一次

  - 一旦正序遍历背包容量，`物品 0` 就会被放入多次 

  - 举个例子：`物品 0` 的重量是 `weight[0] = 1` ，价值 `value[0] = 15` 。如果正序遍历，有以下式子：

    - `dp[1] = dp[1-weight[0]] + value[0] = dp[0] + value[0] = 15`
    - `dp[2] = dp[2-weight[0]] + value[0] = dp[1] + value[0] = 30`

    这表明 `物品 0` 被放入了两次，所以不能正序遍历

  - 倒序遍历时，先算 `dp[2]`

    - `dp[2] = dp[2-weight[0]] + value[0] = 15`
    - `dp[1] = dp[1-weight[0]] + value[0] = 15`

    所以从后往前遍历，每次取得状态不会和之前取得状态重合，这样每种物品就只取了一次

- 一维代码

```c++

int test_2_wei_bag_problem() {
    vector<int> weight = {1, 3, 4};
    vector<int> value = {15, 20, 30};
    int bagweight = 4;

    vector<int> dp(bagweight+1, 0);

    for (int i = 0; i < weight.size(); ++i) {
        for (int j = bagweight; j >= weight[i]; --j) {
            dp[j] = max(dp[j], dp[j-weight[i]] + value[i])
        }
    }
    return dp[bagweight];
}
```



## 4.例题

### 67.分割等和子集（01背包）

给你一个 **只包含正整数** 的 **非空** 数组 `nums` 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

 

**示例 1：**

```
输入：nums = [1,5,11,5]
输出：true
解释：数组可以分割成 [1, 5, 5] 和 [11] 。
```

**示例 2：**

```
输入：nums = [1,2,3,5]
输出：false
解释：数组不能分割成两个元素和相等的子集。
```

 

**提示：**

- `1 <= nums.length <= 200`
- `1 <= nums[i] <= 100`



#### 思路

本题的说法可以转换成：**对于一个只包含正整数的非空数组`nums`，是否能找到一个子集，使得这个子集的元素和等于该数组元素和的一半。**

这就是一个典型的01背包问题。**记数组元素和的一半为`target`**，我们有一个容量为`target`的背包，需要判断能否找到数组的一个子集能够恰好装满这个背包。

仿照着01背包问题，**我们定义`dp[i][j]`为`从数组的(0..i)下标范围内选取若干正整数（可以不选），是否存在一种方案使得选区的正整数和为j`。**



**首先考虑边界情况**

- 如果不选取任何正整数，则被选取的正整数之和等于0。因此对于所有`0 <= i < len`，都有`dp[i][0] = true`
- 当`i == 0`时，只有一个正整数`nums[0]`可以被选，因此`dp[0][nums[0]] = true`



于是我们很容易得到状态转移方程

- 当遍历到的容量`j`大于等于当前数字`num`时，有两种选择，装或者不装。对于这两种选择，只要有一个能够满足，那么该情况就能满足
  - `dp[i][j] = dp[i-1][j] || dp[i-1][j-num]`
- 当遍历到的容量`j`小于当前数字`num`时，不能装入当前数字
  - `dp[i][j] = dp[i-1][j]`

最后返回`dp[len-1][target]`即可

```c++
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int len = nums.size();
        if (len == 1) return false;

        int sum = accumulate(nums.begin(), nums.end(), 0);
        int maxNum = *max_element(nums.begin(), nums.end());

        if (sum % 2 != 0) return false; // 和为奇数，不可能分割
        int target = sum / 2;
        if (maxNum > target) return false; // 最大的数大于target

        // dp[i][j]表示从数组的(0..i)下标范围内选取若干正整数（可以不选）
        // 是否存在一种方案使得选取的正整数和为j
        vector<vector<int>> dp(len, vector<int>(target+1, 0));
        for (int i = 0; i < len; ++i) dp[i][0] = true;
        dp[0][nums[0]] = true;
        for (int i = 1; i < len; ++i) {
            int num = nums[i];
            for (int j = 1; j <= target; ++j) {
                if (j >= num) dp[i][j] = dp[i-1][j] || dp[i-1][j-num];
                else dp[i][j] = dp[i-1][j];
            }
        }
        return dp[len-1][target];
    }
};
```



#### 一维数组优化

```c++
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = 0;
        for (auto i:nums) sum += i;
        if (sum % 2 != 0) return false;
        int target = sum / 2;

        vector<int> dp(target+1); // 背包容量为i，能否装入物品刚好装满背包
        for (int i = 0; i < nums.size(); ++i)
            for (int j = target; j >= nums[i]; --j)
                dp[j] = max(dp[j], dp[j-nums[i]] + nums[i]);
        
        if (dp[target] == target) return true;
        return false;
    }
};
```





### 13.最后一块石头的重量Ⅱ

有一堆石头，用整数数组 `stones` 表示。其中 `stones[i]` 表示第 `i` 块石头的重量。

每一回合，从中选出**任意两块石头**，然后将它们一起粉碎。假设石头的重量分别为 `x` 和 `y`，且 `x <= y`。那么粉碎的可能结果如下：

- 如果 `x == y`，那么两块石头都会被完全粉碎；
- 如果 `x != y`，那么重量为 `x` 的石头将会完全粉碎，而重量为 `y` 的石头新重量为 `y-x`。

最后，**最多只会剩下一块** 石头。返回此石头 **最小的可能重量** 。如果没有石头剩下，就返回 `0`。

 

**示例 1：**

```
输入：stones = [2,7,4,1,8,1]
输出：1
解释：
组合 2 和 4，得到 2，所以数组转化为 [2,7,1,8,1]，
组合 7 和 8，得到 1，所以数组转化为 [2,1,1,1]，
组合 2 和 1，得到 1，所以数组转化为 [1,1,1]，
组合 1 和 1，得到 0，所以数组转化为 [1]，这就是最优值。
```

**示例 2：**

```
输入：stones = [31,26,33,21,40]
输出：5
```

 

**提示：**

- `1 <= stones.length <= 30`
- `1 <= stones[i] <= 100`



#### 思路

**本题的意思是要将原来的石头尽可能分成重量接近的两堆，然后进行粉碎就可以得到最终最小的重量**。

可以通过01背包来解决：定义`dp[i]`为背包容量为`i`的背包能装的最大重量，我们先计算石头的总重量`sum`，然后目标是`target=sum/2`，需要求出容量为`target`的背包所能装的最大重量，求出后将两部分相减即可。

```c++
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        vector<int> dp(15001, 0);
        int sum = 0;
        for (auto i:stones) sum += i;
        int target = sum / 2;
        for (int i = 0; i < stones.size(); ++i)
            for (int j = target; j >= stones[i]; --j)
                dp[j] = max(dp[j], dp[j-stones[i]] + stones[i]);
        return sum - dp[target] - dp[target];
    }
};
```

至于为什么是`sum - dp[target] - dp[target]`，这是因为`target = sum / 2`是向下取整的，一定有`sum - dp[target] >= dp[target]`。



### 14.目标和

给你一个非负整数数组 `nums` 和一个整数 `target` 。

向数组中的每个整数前添加 `'+'` 或 `'-'` ，然后串联起所有整数，可以构造一个 **表达式** ：

- 例如，`nums = [2, 1]` ，可以在 `2` 之前添加 `'+'` ，在 `1` 之前添加 `'-'` ，然后串联起来得到表达式 `"+2-1"` 。

返回可以通过上述方法构造的、运算结果等于 `target` 的不同 **表达式** 的数目。

 

**示例 1：**

```
输入：nums = [1,1,1,1,1], target = 3
输出：5
解释：一共有 5 种方法让最终目标和为 3 。
-1 + 1 + 1 + 1 + 1 = 3
+1 - 1 + 1 + 1 + 1 = 3
+1 + 1 - 1 + 1 + 1 = 3
+1 + 1 + 1 - 1 + 1 = 3
+1 + 1 + 1 + 1 - 1 = 3
```

**示例 2：**

```
输入：nums = [1], target = 1
输出：1
```

 

**提示：**

- `1 <= nums.length <= 20`
- `0 <= nums[i] <= 1000`
- `0 <= sum(nums[i]) <= 1000`
- `-1000 <= target <= 1000`





#### 思路

根据题意，我们需要在每个元素前添加 `+` 或 `-` 。所以我们可以把原集合分为两种集合：**加法集合 `add` **与**减法集合 `sub` **，在加法集合中，所有的元素前添加 `+` ，在减法集合中，所有的元素前添加 `-` 。

假设原集合的所有元素和为 `sum` ，我们有：`add + sub = sum` ，并且有 `add - sub = target` 。所以 `sub = add - target` ，将上式代入第一个式子，得到 `2*add - target = sum` ，于是乎 `add = (target + sum) / 2`。其中，`target` 和 `sum` 都是定值，我们**只需要在原集合中找出一个子集，满足该子集和 `add` 等于 `(target + sum) / 2`** 。

这就转换成了一个01背包问题：给定一个 `nums` 集合，给定一个背包容量 `bagSize = (target + sum) / 2 `，求有多少种组合能够装满容量为`bagSize` 的背包。

定义 `dp[i]` 表示装满容量为 `i` 的背包有多少种方法。那么对于遍历到的数字 `nums[j]` ，有 `dp[i] += dp[i-nums[j]]` ，解释为：装满容量为 `i` 的背包的方法数量可以由装满容量为 `i - nums[j]` 的背包的方法数量推过来。

所以我们需要遍历所有物品，将 `dp[i-nums[j]]` 累加起来，最终得到 `dp[bagSize]` 。



```c++
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum = 0;
        for (auto i : nums) sum += i;
        if (abs(target) > sum) return 0; // target 绝对值比 sum 还大，无解
        if ((target+sum) % 2 == 1) return 0; // 如果无法除尽，表示无解
        int bagSize = (target + sum) / 2;
        vector<int> dp(bagSize + 1, 0);
        dp[0] = 1; // 注意 dp[0] 要初始化为 1 ，否则 dp 数组一直为 0 
        for (int i = 0; i < nums.size(); ++i) {
            for (int j = bagSize; j >= nums[i]; --j) {
                dp[j] += dp[j-nums[i]];
            }
        }
        return dp[bagSize];
    }
};
```





### 17.一和零

给你一个二进制字符串数组 `strs` 和两个整数 `m` 和 `n` 。

请你找出并返回 `strs` 的最大子集的长度，该子集中 **最多** 有 `m` 个 `0` 和 `n` 个 `1` 。

如果 `x` 的所有元素也是 `y` 的元素，集合 `x` 是集合 `y` 的 **子集** 。

 

**示例 1：**

```
输入：strs = ["10", "0001", "111001", "1", "0"], m = 5, n = 3
输出：4
解释：最多有 5 个 0 和 3 个 1 的最大子集是 {"10","0001","1","0"} ，因此答案是 4 。
其他满足题意但较小的子集包括 {"0001","1"} 和 {"10","1","0"} 。{"111001"} 不满足题意，因为它含 4 个 1 ，大于 n 的值 3 。
```

**示例 2：**

```
输入：strs = ["10", "0", "1"], m = 1, n = 1
输出：2
解释：最大的子集是 {"0", "1"} ，所以答案是 2 。
```

 

**提示：**

- `1 <= strs.length <= 600`
- `1 <= strs[i].length <= 100`
- `strs[i]` 仅由 `'0'` 和 `'1'` 组成
- `1 <= m, n <= 100`



#### 思路

dp数组需要从两个维度进行考虑，容量的遍历顺序也是从大往小遍历

```c++
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        // dp[i][j]表示 i 个 0 和 j 个 1 能得到的最长子集长度
        vector<vector<int>> dp(m+1, vector<int>(n+1));
        for (auto str : strs) {
            int zeroNum = 0, oneNum = 0;
            for (auto c : str) {
                if (c == '0') zeroNum++;
                else oneNum++;
            }
            for (int i = m; i >= zeroNum; --i) {
                for (int j = n; j >= oneNum; --j) {
                    dp[i][j] = max(dp[i][j], dp[i-zeroNum][j-oneNum] + 1);
                }
            }
        }
        return dp[m][n];
    }
};
```



做这类题时需要明确题目中具体的定义，根据定义来抽象dp数组的含义

- 纯01背包是求给定背包容量，装满背包的最大价值是多少
- 分割等和子集是求给定背包容量，能不能装满这个背包
- 最后一块石头的重量是求给定背包容量，尽可能装，最多能装多少
- 目标和是求给定背包容量，装满背包有多少种方法
- 一和零是求给定背包容量，装满背包最多有多少个物品

