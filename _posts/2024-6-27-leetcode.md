---
layout:post
title:leetcode记录
description:记录
tag:算法
---
[TOC]

# 1.找出最具竞争力的子序列（单调栈+贪心）

给你一个整数数组 `nums` 和一个正整数 `k` ，返回长度为 `k` 且最具 **竞争力** 的 `nums` 子序列。

数组的子序列是从数组中删除一些元素（可能不删除元素）得到的序列。

在子序列 `a` 和子序列 `b` 第一个不相同的位置上，如果 `a` 中的数字小于 `b` 中对应的数字，那么我们称子序列 `a` 比子序列 `b`（相同长度下）更具 **竞争力** 。 例如，`[1,3,4]` 比 `[1,3,5]` 更具竞争力，在第一个不相同的位置，也就是最后一个位置上， `4` 小于 `5` 。



**示例 1：**

```
输入：nums = [3,5,2,6], k = 2
输出：[2,6]
解释：在所有可能的子序列集合 {[3,5], [3,2], [3,6], [5,2], [5,6], [2,6]} 中，[2,6] 最具竞争力。
```

**示例 2：**

```
输入：nums = [2,4,3,3,5,4,9,6], k = 4
输出：[2,3,3,4]
```



**提示：**

- `1 <= nums.length <= 10^5`
- `0 <= nums[i] <= 10^9`
- `1 <= k <= nums.length`



## 思路

根据题意，最具竞争力的子序列**一定是较小的元素尽可能地放到序列的前面**。所以我们可以遍历一遍数组，维持一个单调栈结构，将较小的元素尽可能地往前放。

那么我们应该如何操作？因为我们可以使用单调栈，所以说我们每次将当前元素和栈顶元素比较，如果**当前元素更小**，并且能够保证**栈内元素数目 + 数组中剩余元素 > k且栈非空**，那么我们就弹出栈顶元素，将当前元素压入栈，一直重复，最后返回栈内自下而上的前k各元素为结果即可。

记nums的大小为n，遍历数组nums，假设当前访问的下标为 i ，对数组nums[i]执行以下操作：

1. 记栈中元素数目为 m，我们不断地进行操作直到不满足以下条件：如果 m > 0且 m + n - i > k且单调栈的栈顶元素大于nums[i]，说明栈顶元素可以被当前数字nums[i]替换，我们就弹出栈顶元素
2. 将nums[i]压入栈中
3. 最后返回栈自下而上的前 k 个元素作为结果

代码如下

```c++
class Solution {
public:
    vector<int> mostCompetitive(vector<int>& nums, int k) {
        vector<int> res;
        int n = nums.size(); // 原数组的大小
        for (int i = 0; i < n; ++i) {
            while (!res.empty() && res.size() + n - i > k && res.back() > nums[i]) {
                res.pop_back();
            }
            res.push_back(nums[i]);
        }
        res.resize(k);
        return res;
    }
};
```





# 2.搜索插入位置（二分搜索）

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。



**示例 1:**

```
输入: nums = [1,3,5,6], target = 5
输出: 2
```

**示例 2:**

```
输入: nums = [1,3,5,6], target = 2
输出: 1
```

**示例 3:**

```
输入: nums = [1,3,5,6], target = 7
输出: 4
```



**提示:**

- `1 <= nums.length <= 10^4`
- `-10^4 <= nums[i] <= 10^4`
- `nums` 为 **无重复元素** 的 **升序** 排列数组
- `-10^4 <= target <= 10^4`



## 思路

不断用二分法逼近查找第一个大于等于target的下标，ans初值设置为数组长度可以省略边界条件的判断，因为存在一种情况是target大于数组中的所有数，此时需要插入到数组长度的位置

```c++
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int left = 0, right = nums.size() - 1;
        int ans = nums.size();
        while (left <= right) {
            // 防止加法溢出，位运算比除法更快
            int middle = ((right - left) >> 1 ) + left;
            if (target > nums[middle]) {
                left = middle + 1;
            } else if (target <= nums[middle]) {
                ans = middle; // 将目标数字插到middle的位置
                right = middle - 1;
            }
        }
        return ans;
    }
};
```





# 3.删除并获得点数（打家劫舍）

给你一个整数数组 `nums` ，你可以对它进行一些操作。

每次操作中，选择任意一个 `nums[i]` ，删除它并获得 `nums[i]` 的点数。之后，你必须删除 **所有** 等于 `nums[i] - 1` 和 `nums[i] + 1` 的元素。

开始你拥有 `0` 个点数。返回你能通过这些操作获得的最大点数。



**示例 1：**

```
输入：nums = [3,4,2]
输出：6
解释：
删除 4 获得 4 个点数，因此 3 也被删除。
之后，删除 2 获得 2 个点数。总共获得 6 个点数。
```

**示例 2：**

```
输入：nums = [2,2,3,3,3,4]
输出：9
解释：
删除 3 获得 3 个点数，接着要删除两个 2 和 4 。
之后，再次删除 3 获得 3 个点数，再次删除 3 获得 3 个点数。
总共获得 9 个点数。
```



**提示：**

- `1 <= nums.length <= 2 * 10^4`
- `1 <= nums[i] <= 10^4`



## 思路

题目中的条件：“**每次操作中，选择任意一个 `nums[i]` ，删除它并获得 `nums[i]` 的点数。之后，你必须删除 所有 等于 `nums[i] - 1` 和 `nums[i] + 1` 的元素**”，可以理解为打家劫舍。

记元素`x`在数组中出现的次数为`cx`，我们可以用一个数组`sum`记录数组`nums`中**所有相同元素之和**，即`sum[x] = x * cx`。若选择了`x`，则可以获取`sum[x]`的点数，且无法再选择`x-1`和`x+1`。

```c++
class Solution {
public:
    int rob(vector<int> &sum) {
        int size = sum.size();
        int dp[size];
        dp[0] = sum[0];
        dp[1] = max(sum[0], sum[1]);
        for (int i = 2; i < size; ++i) {
            dp[i] = max(dp[i - 1], dp[i - 2] + sum[i]);
        }
        return dp[size - 1];
    }

    int deleteAndEarn(vector<int>& nums) {
        int maxVal = 0;
        for (auto val : nums) maxVal = max(maxVal, val);
        vector<int> sum(maxVal + 1);
        for (auto val : nums) sum[val] += val;
        return rob(sum);
    }
};
```

由于在打家劫舍中，每个变量的状态只与前两个变量的状态有关，所以可以不使用数组，只用两个变量来实现。

```c++
class Solution {
public:
    int rob(vector<int> &nums) {
        int size = nums.size();
        int first = nums[0], second = max(nums[0], nums[1]);
        for (int i = 2; i < size; i++) {
            int temp = second;
            second = max(first + nums[i], second);
            first = temp;
        }
        return second;
    }

    int deleteAndEarn(vector<int>& nums) {
        int maxVal = 0;
        for (auto val : nums) maxVal = max(maxVal, val);
        vector<int> sum(maxVal + 1);
        for (auto val : nums) sum[val] += val;
        return rob(sum);
    }
};
```





# 4.K 秒后第 N 个元素的值（前缀和）

给你两个整数 `n` 和 `k`。

最初，你有一个长度为 `n` 的整数数组 `a`，对所有 `0 <= i <= n - 1`，都有 `a[i] = 1` 。每过一秒，你会同时更新每个元素为其前面所有元素的和加上该元素本身。例如，一秒后，`a[0]` 保持不变，`a[1]` 变为 `a[0] + a[1]`，`a[2]` 变为 `a[0] + a[1] + a[2]`，以此类推。

返回 `k` 秒后 `a[n - 1]` 的**值**。

由于答案可能非常大，返回其对 `109 + 7` **取余** 后的结果。



**示例 1：**

**输入：**n = 4, k = 5

**输出：**56

**解释：**

| 时间（秒） | 数组状态    |
| ---------- | ----------- |
| 0          | [1,1,1,1]   |
| 1          | [1,2,3,4]   |
| 2          | [1,3,6,10]  |
| 3          | [1,4,10,20] |
| 4          | [1,5,15,35] |
| 5          | [1,6,21,56] |

**示例 2：**

**输入：**n = 5, k = 3

**输出：**35

**解释：**

| 时间（秒） | 数组状态       |
| ---------- | -------------- |
| 0          | [1,1,1,1,1]    |
| 1          | [1,2,3,4,5]    |
| 2          | [1,3,6,10,15]  |
| 3          | [1,4,10,20,35] |



**提示：**

- `1 <= n, k <= 1000`



## 思路

注意每次计算前缀和后取余，不要最后返回结果的时候取余，不然会爆longlong。还有一点就是不能暴力遍历，会超时，所以需要用dp

```c++
class Solution {
public:
    int valueAfterKSeconds(int n, int k) {
        int mod = 1e9 + 7;
        long long a[n];
        for (int i = 0; i < n; ++i) a[i] = 1;
        while (k--) {
            long long sum[1005] = {1};

            for (int i = 1; i < n; ++i) {
                sum[i] = sum[i - 1] + a[i];
                sum[i] %= mod;
            }

            for (int i = 0; i < n; ++i) {
                a[i] = sum[i];
            }
        }
        return a[n - 1];
    }
};
```





# 5.救生艇（双指针+贪心）

给定数组 `people` 。`people[i]`表示第 `i` 个人的体重 ，**船的数量不限**，每艘船可以承载的最大重量为 `limit`。

每艘船最多可同时载两人，但条件是这些人的重量之和最多为 `limit`。

返回 *承载所有人所需的最小船数* 。



**示例 1：**

```
输入：people = [1,2], limit = 3
输出：1
解释：1 艘船载 (1, 2)
```

**示例 2：**

```
输入：people = [3,2,2,1], limit = 3
输出：3
解释：3 艘船分别载 (1, 2), (2) 和 (3)
```

**示例 3：**

```
输入：people = [3,5,3,4], limit = 5
输出：4
解释：4 艘船分别载 (3), (3), (4), (5)
```



**提示：**

- `1 <= people.length <= 5 * 10^4`
- `1 <= people[i] <= limit <= 3 * 10^4`



## 思路

本题是较明显的贪心问题。要承载所有人所需的船数尽可能的小，就需要每艘船装的人尽量多。怎么装才能使人尽量多呢？自然而然是**把体重大的和体重小的安排在同一艘船上**。所以贪心选择策略是：**每艘船装当前最大体重和最小体重的人，如果总体重超过`limit`，则大体重的自己上一艘船，小体重和下一个大体重的匹配**。实现这个策略可以使用**双指针**，在此之前我们需要`people`数组有序。

```c++
class Solution {
public:
    int numRescueBoats(vector<int>& people, int limit) {
        int ans = 0;
        int num = people.size();
        sort(people.begin(), people.end(), greater<int>());
        int left = 0, right = num - 1;
        while (left <= right) {
            ans++;
            if (left == right) break;
            if (people[left] + people[right] <= limit) {
                left++, right--;
            } else {
                left++;
            }
        }
        return ans;
    }
};
```





# 6.甲板上的战舰

给你一个大小为 `m x n` 的矩阵 `board` 表示甲板，其中，每个单元格可以是一艘战舰 `'X'` 或者是一个空位 `'.'` ，返回在甲板 `board` 上放置的 **战舰** 的数量。

**战舰** 只能水平或者垂直放置在 `board` 上。换句话说，战舰只能按 `1 x k`（`1` 行，`k` 列）或 `k x 1`（`k` 行，`1` 列）的形状建造，其中 `k` 可以是任意大小。两艘战舰之间至少有一个水平或垂直的空位分隔 （即没有相邻的战舰）。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/04/10/battelship-grid.jpg)

```
输入：board = [["X",".",".","X"],[".",".",".","X"],[".",".",".","X"]]
输出：2
```

**示例 2：**

```
输入：board = [["."]]
输出：0
```



**提示：**

- `m == board.length`
- `n == board[i].length`
- `1 <= m, n <= 200`
- `board[i][j]` 是 `'.'` 或 `'X'`



## 思路

遍历一次board，找到一个`X`就遍历行和列，将`X`改为`.`，统计答案即可

```c++
class Solution {
public:
    int countBattleships(vector<vector<char>>& board) {
        int row = board.size();
        int col = board[0].size();
        int ans = 0;

        for (int i = 0; i < row; ++i) {
            for (int j = 0; j < col; ++j) {
                if (board[i][j] == 'X') {
                    board[i][j] = '.';
                    // 遍历列
                    for (int k = j + 1; k < col && board[i][k] == 'X'; ++k)
                        board[i][k] = '.';
                    // 遍历行
                    for (int k = i + 1; k < row && board[k][j] == 'X'; ++k)
                        board[k][j] = '.';
                    ans++;
                }
            }
        }
        return ans;
    }
};
```





# 7.不同路径Ⅱ

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish”）。

现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？

网格中的障碍物和空位置分别用 `1` 和 `0` 来表示。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/04/robot1.jpg)

```
输入：obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]
输出：2
解释：3x3 网格的正中间有一个障碍物。
从左上角到右下角一共有 2 条不同的路径：
1. 向右 -> 向右 -> 向下 -> 向下
2. 向下 -> 向下 -> 向右 -> 向右
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/11/04/robot2.jpg)

```
输入：obstacleGrid = [[0,1],[0,0]]
输出：1
```



**提示：**

- `m == obstacleGrid.length`
- `n == obstacleGrid[i].length`
- `1 <= m, n <= 100`
- `obstacleGrid[i][j]` 为 `0` 或 `1`



## 思路

这是一个带障碍物的二维dp，我们可以将其降到一维，代码如下

```c++
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int n = obstacleGrid.size(), m = obstacleGrid.at(0).size();
        vector<int> dp(m); // 用一维数组遍历整个棋盘

        if (obstacleGrid[0][0] == 0) dp[0] = 1;
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < m; ++j) {
                if (obstacleGrid[i][j] == 1) {
                    dp[j] = 0;
                    continue;
                }
                if (j - 1 >= 0 && obstacleGrid[i][j - 1] == 0) {
                    dp[j] += dp[j - 1]; // 这里的dp[j]是继承了上一行的dp[j]，所以只用加上前一个dp[j - 1]即可
                }
            }
        }
        return dp.back();
    }
};
```



二维做法如下

```c++
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size(), n = obstacleGrid.at(0).size();
        int dp[105][105] = {0};
        if (obstacleGrid[0][0] == 0) dp[0][0] = 1; // 特判一下起点

        for (int i = 1; i < n; ++i) {
            if (obstacleGrid[0][i] != 1) dp[0][i] += dp[0][i - 1];
            else dp[0][i] = 0;
        }
        for (int j = 1; j < m; ++j) {
            if (obstacleGrid[j][0] != 1) dp[j][0] += dp[j - 1][0];
            else dp[j][0] = 0;
        }
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                if (obstacleGrid[i][j] == 1) dp[i][j] = 0;
                else dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        return dp[m - 1][n - 1];
    }
};
```





# 8.施咒的最大总伤害（dp）

一个魔法师有许多不同的咒语。

给你一个数组 `power` ，其中每个元素表示一个咒语的伤害值，可能会有多个咒语有相同的伤害值。

已知魔法师使用伤害值为 `power[i]` 的咒语时，他们就 **不能** 使用伤害为 `power[i] - 2` ，`power[i] - 1` ，`power[i] + 1` 或者 `power[i] + 2` 的咒语。

每个咒语最多只能被使用 **一次** 。

请你返回这个魔法师可以达到的伤害值之和的 **最大值** 。



**示例 1：**

**输入：**power = [1,1,3,4]

**输出：**6

**解释：**

可以使用咒语 0，1，3，伤害值分别为 1，1，4，总伤害值为 6 。

**示例 2：**

**输入：**power = [7,1,6,6]

**输出：**13

**解释：**

可以使用咒语 1，2，3，伤害值分别为 1，6，6，总伤害值为 13 。



**提示：**

- `1 <= power.length <= 10^5`
- `1 <= power[i] <= 10^9`



## 思路

本题为动态规划，每个咒语都有一个对应的伤害值，选了某个咒语后，该咒语的伤害值+-2范围内的咒语都不能选，给定一个咒语数组，要我们求最大能够造成多少伤害。注意若有多个相同伤害值的咒语，可以重复选择。所以对于每种伤害值的咒语，他能打出的伤害应该是`ai*bi`，**其中`ai`表示咒语的伤害值，`bi`表示该伤害值咒语的数量。**

**我们用`f[i]`来记录最后使用第`i`个咒语能打出的最大总伤害**，我们先根据咒语的本身基础伤害排序，那么对于`j<i`，有下面的状态转移方程：`f[i] = f[j] + ai*bi`，其中要满足`aj < ai - 2`。

```c++
class Solution {
public:
    long long maximumTotalDamage(vector<int>& power) {
        map<int, int>mp; // 统计每种伤害值咒语的个数
        for (auto x : power) mp[x]++;

        typedef pair<int, int> pii;
        // vec里按伤害从小到大保存每种咒语的伤害和数量
        vector<pii> vec;
        vec.push_back(pii(-1e9, 0));
        for (auto &p : mp) vec.push_back(p);

        int n = vec.size();
        // f[i]表示最后释放了第i种咒语的最大总伤害
        long long f[n];
        f[0] = 0;

        long long mx = 0;
        // 计算释放第i个咒语能获得的最大伤害
        for (int i = 1, j = 1; i < n; ++i) {
            // 求当前满足条件的最大伤害
            while (j < i && vec[j].first < vec[i].first - 2) {
                mx = max(mx, f[j]);
                ++j;
            }
            // 释放第i个咒语的伤害+之前能达到的最大伤害 = 释放第i个咒语所能达到的最大伤害
            f[i] = mx + 1LL * vec[i].first * vec[i].second;
        }

        long long ans = 0;
        // 求最大值返回
        for (int i = 1; i < n; ++i) ans = max(ans, f[i]);
        return ans;
    }
};
```





# 9.构成整天的下标对数目（哈希表）

给你一个整数数组 `hours`，表示以 **小时** 为单位的时间，返回一个整数，表示满足 `i < j` 且 `hours[i] + hours[j]` 构成 **整天** 的下标对 `i`, `j` 的数目。

**整天** 定义为时间持续时间是 24 小时的 **整数倍** 。

例如，1 天是 24 小时，2 天是 48 小时，3 天是 72 小时，以此类推。



**示例 1：**

**输入：** hours = [12,12,30,24,24]

**输出：** 2

**解释：**

构成整天的下标对分别是 `(0, 1)` 和 `(3, 4)`。

**示例 2：**

**输入：** hours = [72,48,24,3]

**输出：** 3

**解释：**

构成整天的下标对分别是 `(0, 1)`、`(0, 2)` 和 `(1, 2)`。



**提示：**

- `1 <= hours.length <= 5 * 10^5`
- `1 <= hours[i] <= 10^9`



## 思路

看数据范围直接进行枚举会超时，这里采用哈希表查找，本题的思路和两数之和差不多。

我们遍历一遍数组，**计算当前遍历到的元素对24的余数p，那么他所需要配对的数字的余数就应该是(24 - p) % 24**，记录配对的次数即可

```c++
class Solution {
public:
    long long countCompleteDayPairs(vector<int>& hours) {
        unordered_map <int, int> map_cnt;
        long long ans = 0;
        for (auto hour : hours) {
            int p = hour % 24; // 当前的余数
            int q = (24 - p) % 24; // 需要的余数
            if (map_cnt.find(q) != map_cnt.end()) {
                ans += map_cnt[q];
            }
            map_cnt[p]++;
        }
        return ans;
    }
};
```





# 10.最大正方形

在一个由 `'0'` 和 `'1'` 组成的二维矩阵内，找到只包含 `'1'` 的最大正方形，并返回其面积。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/26/max1grid.jpg)

```
输入：matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
输出：4
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/11/26/max2grid.jpg)

```
输入：matrix = [["0","1"],["1","0"]]
输出：1
```

**示例 3：**

```
输入：matrix = [["0"]]
输出：0
```



**提示：**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 300`
- `matrix[i][j]` 为 `'0'` 或 `'1'`



## 思路

我们定义`dp[i][j]`表示以`(i, j)`为右下角，且只包含1的正方形边长的最大值。

- 如果该位置的值为0，则`dp[i][j] = 0`，因为当前位置不可能在由1组成的正方形中

- 如果该位置为1，则`dp[i][j]`的值由其**上方、左方和左下方**的三个相邻的`dp`值决定，**具体而言，当前位置的元素值等于三个相邻位置的元素中的最小值加1**，状态转移方程如下
  - `dp[i][j] = min({dp[i-1][j], dp[i][j-1], dp[i-1][j-1]}) + 1`

**下面给出证明**

![1277-1.png](https://pic.leetcode-cn.com/14aa58be2ea5c9b36a722db76d2e843c4c909e312223a8461a3d2d93bc734b42-1277-1.png)

如上图所示，若对于`(i, j)`有`dp[i][j] = 4`，我们将以`(i, j)`为右下角、边长为`4`的正方形涂上颜色，可以发现其左侧位置`(i, j-1)`，上方位置`(i-1, j)`，左上方位置`(i-1, j-1)`均可以作为一个边长为`4 - 1 = 3`的正方形的右下角。也就是说，**这些位置的`dp`值至少为`3`**，即：

- `dp[i][j-1] >= dp[i][j] - 1`
- `dp[i-1][j] >= dp[i][j] - 1`
- `dp[i-1][j-1] >= dp[i][j] - 1`

联立以上三个不等式，得到：

- `min({dp[i][j-1], dp[i-1][j], dp[i-1][j-1]}) >= dp[i][j] - 1`

**以上推导过程是我们通过固定`dp[i][j]`的值，判断其相邻位置与之的关系得到的不等式。同理，我们也能固定`dp[i][j]`相邻位置的值，得到另外的限制条件。**



![1277-2.png](https://pic.leetcode-cn.com/dfcaa5390732f1b9d1d9c71f696957cbc91904c52e23705a061bae727228126d-1277-2.png)

如上图所示，假设`dp[i][j-1]`，`dp[i-1][j]`和`dp[i-1][j-1]`中最小值为`3`，

也就是说`(i, j-1)`、`(i-1, j)`和`(i-1, j-1)`均可以作为一个边长为`3`的正方形的右下角。

我们将这些边长为`3`的正方形依次涂上色，可以发现如果`(i, j)`的元素为`1`，那么它可以作为一个边长为`4`的正方形的右下角，`dp`值至少为`4`，即：

- `dp[i][j] >= min({{dp[i][j-1], dp[i-1][j], dp[i-1][j-1]}}) + 1`



联立两种情况得到的不等式，可以得到状态转移方程

- `dp[i][j] = min({{dp[i][j-1], dp[i-1][j], dp[i-1][j-1]}}) + 1`

```c++
class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        if (matrix.size() == 0 || matrix[0].size() == 0) return 0;

        int ans = 0;
        int row = matrix.size(), col = matrix[0].size();
        vector<vector<int>> dp(row, vector<int>(col));
        for (int i = 0; i < row; ++i) {
            for (int j = 0; j < col; ++j) {
                if (matrix[i][j] == '1') {
                    if (i == 0 || j == 0) dp[i][j] = 1;
                    else dp[i][j] = min({dp[i-1][j], dp[i][j-1], dp[i-1][j-1]}) + 1;
                }
                ans = ans > dp[i][j] ? ans : dp[i][j];
            }
        }
        return ans * ans;
    }
};
```
