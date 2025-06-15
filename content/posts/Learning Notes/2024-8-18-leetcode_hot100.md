---
layout: post
title: leetcode hot100
description: 记录
tag: 算法
---

[TOC]



## 1.两数之和

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 **和为目标值** *`target`* 的那 **两个** 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

**示例 1：**

```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```

**示例 2：**

```
输入：nums = [3,2,4], target = 6
输出：[1,2]
```

**示例 3：**

```
输入：nums = [3,3], target = 6
输出：[0,1]
```

**提示：**

- `2 <= nums.length <= 10^4`
- `-10^9 <= nums[i] <= 10^9`
- `-10^9 <= target <= 10^9`
- **只会存在一个有效答案**

 

**进阶：**你可以想出一个时间复杂度小于 `O(n^2)` 的算法吗？



### 思路

**对于每一个遍历到的 x，我们只需要找到相应的 (target - x) 即可。**于是我们可以创建一个哈希表，只需要对 nums 数组进行一次遍历，每次遍历到 x，查询当前哈希表中是否存在 (target - x)，**假设没有找到，那么将 x 插入到哈希表中**，保证 x 不会与自己匹配。代码如下：

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        // 创建哈希表
        unordered_map<int, int> hashtable;
        // 只用遍历一次数组
        for (int i = 0; i < nums.size(); ++i) {
            // 尝试寻找目标元素
            auto it = hashtable.find(target - nums[i]);
            // 找到了，返回答案
            if (it != hashtable.end()) {
                return {it->second, i};
            }
            // 将x插入到哈希表中，保证不会与自己匹配
            hashtable[nums[i]] = i;
        }
        return {};
    }
};
```



## 2.字母异位词分组（字符串排序）

给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。

**字母异位词** 是由重新排列源单词的所有字母得到的一个新单词。

**示例 1:**

```
输入: strs = ["eat", "tea", "tan", "ate", "nat", "bat"]
输出: [["bat"],["nat","tan"],["ate","eat","tea"]]
```

**示例 2:**

```
输入: strs = [""]
输出: [[""]]
```

**示例 3:**

```
输入: strs = ["a"]
输出: [["a"]]
```

**提示：**

- `1 <= strs.length <= 10^4`
- `0 <= strs[i].length <= 100`
- `strs[i]` 仅包含小写字母





### 思路

由于互为字母异位词的两个字符串包含的字母相同，因此我们只需要**对每次遍历到的字符串进行排序**，就可以知道哪些字符串互为字母异位词。对于这种思路，我们**可以用每一组字母异位词排序后的字符串作为哈希表的键**，**每个字符串作为该键对应的值**。代码如下：

```c++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        // string是每一组字母异位词排序后的结果，将其设置为哈希表的键
        unordered_map<string, vector<string>> mp;
        for (string& str: strs) {
            // 创建字符串key存放排序后的结果
            string key = str;
            sort(key.begin(), key.end());
            // 向哈希表的该键插入值，创建对应关系
            mp[key].push_back(str);
        }

        vector<vector<string>> ans;
        for (auto it = mp.begin(); it != mp.end(); it++) {
            // 将哈希表的值压入vector
            ans.push_back(it->second);
        }
        return ans;
    }
};
```



## 3.最长连续序列

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

**示例 1：**

```
输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

**示例 2：**

```
输入：nums = [0,3,7,2,5,8,4,6,0,1]
输出：9
```

**提示：**

- `0 <= nums.length <= 10^5`
- `-10^9 <= nums[i] <= 10^9`



### 思路

根据题意，给我们一个未排序的整数数组，要求找到该整数数组中最长的连续序列。所谓连续序列，就是形如1、2、3、4、5……的序列，**序列中任意两个相邻的数字相差1**。

先对给定的数组进行排序，然后用双指针的方法，**令两个指针分别指向前后两个相邻元素**，遍历一遍这个数组。**若遍历到的两个相邻元素相差1**，那么答案就加1，**否则判断当前遍历到的两个相邻元素是相等还是相差超过1**，**若相等**，则直接将两个指针往后移，不用将当前答案归零，**否则就需要先将答案归零**，再将两个指针向后移。

需要注意特判一下空数组的情况，具体代码如下：

```c++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if (nums.size() == 0) return 0; // 特判空数组
        int ans = 1; // 记录最终答案
        int tmp = 1; // 记录当前答案
        sort(nums.begin(), nums.end());

        int i = 0, j = i + 1; // 双指针
        while (j < nums.size()) { // 遍历数组
            // 若相差1，则当前答案++，指针往后移
            if (nums[j] - nums[i] == 1) {
                tmp++;
                i = j++;
            } else { // 若相差不为1，则需要判断是相等还是相差超过1
                // 先更新答案，再判断要不要将当前tmp归零
                if (tmp > ans) ans = tmp;
                // 若相等，则只用把两个指针往后挪
                if (nums[j] == nums[i]) {
                    i = j++;
                    continue;
                }
                // 否则将tmp归零，再将指针往后挪
                tmp = 1;
                i = j++;
            }
        }
        // 最后再更新一次答案
        if (tmp > ans) ans = tmp;
        return ans;
    }
};
```

另解：哈希表

1. 首先，将数组中的所有整数存入哈希集合中，这样可以在接近常数时间内进行查找操作，同时去除重复的整数。
2. 然后，遍历哈希集合中的每个整数。如果当前整数的前一个整数不在集合中，说明当前整数可能是一个连续序列的起点；如果前一个整数存在，那这个数肯定不是开头，直接跳过。
3. 从可能的起点开始，不断检查下一个连续的整数是否在集合中，如果在，则继续延伸连续序列，同时记录连续序列的长度。
4. 最后，在遍历过程中不断更新最长连续序列的长度。

```c++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> st(nums.begin(), nums.end());
        int res = 0;
        // 遍历哈希集合而不是 nums 
        // 如果测试用例为 [1,1,1,…,1,2,3,4,5,…]，遍历 nums 的做法会导致每一个 1 都跑一个 O(n) 的循环
        for (auto& x : st) {
            if (st.contains(x - 1)) {
                continue;
            }
            int y = x + 1;
            while (st.contains(y)) {
                ++y;
            }
            res = max(res, y - x);
        }
        return res;
    }
};
```





## 4.移动零（双指针）

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

**示例 1:**

```
输入: nums = [0,1,0,3,12]
输出: [1,3,12,0,0]
```

**示例 2:**

```
输入: nums = [0]
输出: [0]
```

**提示**:

- `1 <= nums.length <= 10^4`
- `-2^31 <= nums[i] <= 2^31 - 1`



### 思路

定义两个指针，左指针指向**当前已经处理好的序列的尾部**，右指针指向**待处理序列的头部**。我们将右指针不断向右移动，**每次右指针指向非零数时，就将左右指针对应的数交换，同时左指针向右移动**。

通过以上操作，我们能够减少交换次数，且整个数组我们只需要遍历一遍，大大的优化了暴力解法遍历多次所消耗的时间复杂度。

那么这个过程是如何操作的呢？我们以下面的输入输出为例

```
输入: nums = [0,1,0,3,12]
输出: [1,3,12,0,0]
```

开始时左右指针都指向0，由于右指针 right 指向的是0，所以不进行交换操作，直接将 right 往后移。此时 left 指向0，right 指向1，right 指向了非零数，将0和1进行交换，并且 left 往后移。此时，**我们就说数组前两位已经排好序**，也就是前面所说的**左指针指向当前已经处理好的序列的尾部，右指针指向待处理序列的头部**。根据这个可以知道，left 始终指向0，且该0是已经处理好的序列的最后一位。于是不断通过这个操作，直到 right 移动到数组的末尾即可。

那么我们再考虑特殊情况，假设他给定的数组里没有0，那么通过以上操作可以知道，左右指针始终指向同一个数字，即使进行交换操作也是这个数字自己和自己交换，并不会影响数组的顺序。具体代码如下：

```c++
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int n = nums.size(), left = 0, right = 0;
        while (right < n) {
            if (nums[right]) {
                swap(nums[left], nums[right]);
                left++;
            }
            right++;
        }
    }
};
```



## 5.盛水最多的容器（双指针）

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

**说明：**你不能倾斜容器。

**示例 1：**

![1.png](https://s2.loli.net/2024/11/24/8eQjdGvJAyDHN74.png)

```
输入：[1,8,6,2,5,4,8,3,7]
输出：49 
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
```

**示例 2：**

```
输入：height = [1,1]
输出：1
```

**提示：**

- `n == height.length`
- `2 <= n <= 10^5`
- `0 <= height[i] <= 10^4`





### 思路

我们对这个过程进行分析，**容器的盛水量与双指针指向的元素中的较小者，以及双指针之间的距离有关**。为了优化上面算法的时间复杂度，我们必须减少循环嵌套。因此我们可以先将两个指针分别指向数组的头和尾，**将两个指针向内移动**，直到两个指针相遇，找出最大容量，这样做就可以只用一层循环解决。

为了更低的时间复杂度，我们需要每次移动指针时**保证盛水量尽量不减**。我们把双指针指向的元素当作两个木板：

当我们**向内移动短板时**，**(i - j) 一定减小，但是 min(height[i], height[j]) 可能增大，所以最终结果可能增大；**

当我们**向内移动长板时，(i - j) 一定减小，min(height[i], height[j]) 要么不变，要么减小，所以最终结果一定减小。**

根据以上分析，我们每次向内移动短板，并更新答案即可。具体代码如下：

```c++
class Solution {
public:
    int maxArea(vector<int>& height) {
        int len = height.size(); // 数组长度
        int i = 0, j = len - 1; // 定义双指针，指向起点与终点
        int ans = 0; // 记录答案

        while (i < j) {
            int tmp = min(height[i], height[j]) * (j - i);
            if (tmp > ans) ans = tmp;
            // 每次移动短板
            if (height[i] > height[j]) {
                j--;
            } else {
                i++;
            }
        }
        return ans;
    }
};
```



## 6.三数之和

给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请

你返回所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

**示例 1：**

```
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
解释：
nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
注意，输出的顺序和三元组的顺序并不重要。
```

**示例 2：**

```
输入：nums = [0,1,1]
输出：[]
解释：唯一可能的三元组和不为 0 。
```

**示例 3：**

```
输入：nums = [0,0,0]
输出：[[0,0,0]]
解释：唯一可能的三元组和为 0 。
```

 

**提示：**

- `3 <= nums.length <= 3000`
- `-10^5 <= nums[i] <= 10^5`



### 思路

因为要找一个三元组，所以开始的想法是用三个for循环进行枚举，当找到满足和为0的三元组就将其压入答案vector中。但这样会有问题，比如示例1中，我们用三重for循环枚举，前两重循环分别指向第一个元素-1和第二个元素0，第三重循环往后遍历时，会找到满足条件的1，**此时得到满足条件的三元组[-1,0,1]**。而当前两重循环遍历到第二个元素0和第三个元素1时，第三重循环会遍历到-1，也满足条件，但这个三元组与前面的三元组[-1,0,1]重复了。若直接在三重循环的基础上进行去重，还要用到哈希表，这样时间复杂度与空间复杂度都会比较高，因此考虑其他做法。

### 如何避免重复问题

因为上述做法出现了三元组重复的问题，即[-1,0,1]、[0,-1,1]、[1,0,-1]。那么有没有一种策略能够避免这种重复问题呢？答案当时是可以。我们只需要对于给定的数组进行排序，再通过以上三重循环思路进行枚举，**还需要注意的是枚举时需要判断当前循环指向的元素与上一个元素是否相同，若相同则直接跳过。**

### 时间复杂度的进一步优化

即使采用了上述做法避免了重复问题，我们仍然没有跳出三重循环的束缚，时间复杂度仍然为O(N^3)。我们要对其进行优化，可以考虑内层的两个循环。当我们确定最外层循环的数字 a 时，需要对内两层循环 b 和 c 进行枚举。

由于此时数组已经经过排序，所以对 b 和 c 的变化情况进行分析：首先 c 一定始终在 b 右侧，当 b 向右移动(b 增大)时，c 一定得变小，这是因为 a 不变，我们要找到 (b + c = -a) 的情况。所以说内两层循环可以进行合并操作，b 从左往右遍历的同时，c 从右往左遍历。具体看以下代码：

```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        int len = nums.size();
        // 先对数组进行排序，避免后续枚举时计算重复三元组
        sort(nums.begin(), nums.end());
        vector<vector<int>> ans;

        // 第一重循环枚举a
        for (int first = 0; first < len; ++first) {
            // 先判断当前枚举到的元素是否与上一个元素相同，避免重复
            if (first > 0 && nums[first] == nums[first - 1]) continue;

            // 指向c的指针，开始初始化在最右端
            int third = len - 1;
            int target = -nums[first];

            // 第二重循环枚举b
            for (int second = first + 1; second < len; ++second) {
                // 先判断当前枚举到的元素是否与上一个元素相同，避免重复
                if (second > first + 1 && nums[second] == nums[second - 1]) continue;
                
                // 保证b始终在c左侧
                while (second < third && nums[second] + nums[third] > target)
                    --third;

                // 指针相遇就枚举完了
                if (second == third) break;
                
                // 找到一组解
                if (nums[second] + nums[third] == target)
                    ans.push_back({nums[first], nums[second], nums[third]});
            }
        }
        return ans;
    }
};
```



## 7.接雨水（dp）

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

**示例 1：**

![image.png](https://s2.loli.net/2025/02/14/t1AysBgIP7MTVvm.png)

```
输入：height = [0,1,0,2,1,0,1,3,2,1,2,1]
输出：6
解释：上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）。 
```

**示例 2：**

```
输入：height = [4,2,0,3,2,5]
输出：9
```

 

**提示：**

- `n == height.length`
- `1 <= n <= 2 * 10^4`
- `0 <= height[i] <= 10^5`



### 思路

根据题意，给定的高度数组最左边和最右边的点一定接不了雨水，所以我们需要求出第1个点到第(len - 1)个点所能存储的雨水之和。

而如何求每个点能接到的雨水数量，**取决于该点左边最大高度与右边最大高度的最小值和当前点的高度**。所以最开始的思路是最外层用一个 for 循环遍历从 1 到 (len - 1) 的数组元素，对于每个元素再通过内层两个for循环求出左边的最大高度与右边的最大高度，然后用其中较小者减去当前高度即可。由于嵌套了两层循环，时间复杂度为 O(n^2)，于是考虑优化。

多出来的一层循环用来求每个点左边最高与右边最高，我们可以通过 dp 事先求出每个点的左边最高与右边最高，然后只用遍历一次数组就可求得答案，具体代码如下：

```c++
class Solution {
public:
    int trap(vector<int>& height) {
        int len = height.size(); // 记录数组长度
        int ans = 0;
        vector<int> left_max(len); // 左侧柱子最大值
        vector<int> right_max(len); // 右侧柱子最大值

        // 初始化dp数组
        left_max[0] = height[0];
        right_max[len - 1] = height[len - 1];

        for (int i = 1; i < len; i++) {
            left_max[i] = max(left_max[i - 1], height[i]);
        }
        for (int i = len - 2; i >= 0; i--) {
            right_max[i] = max(right_max[i + 1], height[i]);
        }

        for (int i = 0; i < len; i++) {
        	// 判断是否为负数，非负就累加到答案
            if (min(left_max[i], right_max[i]) - height[i] >= 0)
                ans += min(left_max[i], right_max[i]) - height[i];
        }
        return ans;
    }
};
```



## 8.无重复的最长字串

给定一个字符串 `s` ，请你找出其中不含有重复字符的最长子串的长度。

**示例 1:**

```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

**示例 3:**

```
输入: s = "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串
```

**提示：**

- `0 <= s.length <= 5 * 10^4`
- `s` 由英文字母、数字、符号和空格组成



### 思路

很明显的一道滑动窗口。我们定义两个指针：慢指针 i 和快指针 j，i 指向滑动窗口的第一个元素，j 指向待添加元素。每次挪动 i 指针或 j 指针后，用 k 指针从 i 指针所指位置往后遍历，一直到 j 指针所指位置的前面一个位置，判断 k 遍历到的字符与 j 是否相同，再根据判断结果挪动对应的指针即可。

**注意特判空字符串与 j 越界后跳出循环的情况。**

代码如下

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if (s.length() == 0) return 0;
        int ans = 1;
        int len = s.length(); // 字符串长度
        int i = 0, j = 1; // 双指针滑动窗口

        while (i < len - 1) {
            // 每次都从当前窗口第一个字母遍历一遍，看是否有重复
            bool flag = true; // 判断是否有重复
            int k;
            for (k = i; k < j; ++k) {
                if (s[j] == s[k]) {
                    flag = false;
                    break;
                }
            }
            // 若无重复，则更新答案
            if (flag) {
                if (j - i + 1 > ans) ans = j - i + 1;
                j++; //快指针往后挪
            } else { // 若有重复，则移动慢指针
                i = k + 1;
            }
            if (j > len - 1) break;
        }
        return ans;
    }
};
```



### 优化（？）

因为在上述代码中的 while 循环里，我们是通过 for 循环遍历一遍当前不重复字符串来判断 j 指针指向的字符是否重复，造成了时间上的一次浪费。实际上，我们可以用哈希集合来代替这个循环。

具体代码如下

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        if (s.empty()) return 0;
        int ans = 1;
        int len = s.length(); // 字符串长度
        int i = 0, j = 0; // 滑动窗口的左右指针
        unordered_set<char> charSet; // 存储窗口中的字符

        while (i < len && j < len) {
            if (charSet.find(s[j]) == charSet.end()) { // 当前字符不在窗口中
                charSet.insert(s[j++]); // 将当前字符加入窗口，并向右移动窗口
                ans = max(ans, j - i); // 更新最大长度
            } else { // 当前字符在窗口中
                charSet.erase(s[i++]); // 移除窗口左边界的字符，并向右移动左指针
            }
        }
        return ans;
    }
};
```

**上一个做法提交运行时间平均在10ms左右，这个做法运行时间平均在27ms，是否优化了还存疑**



## 9.找到字符串中所有字母异位词

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

**异位词** 指由相同字母重排列形成的字符串（包括相同的字符串）。

**示例 1:**

```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
```

 **示例 2:**

```
输入: s = "abab", p = "ab"
输出: [0,1,2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。
```

 

**提示:**

- `1 <= s.length, p.length <= 3 * 10^4`
- `s` 和 `p` 仅包含小写字母



### 超时代码

用滑动窗口去遍历整个字符串，每次遍历到一个字串就sort一次，再与sort好的字符串 p 比较，当字符串长度过长时，这个做法会超时.

```c++
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        vector<int> ans;
        int len_s = s.length(); // s的长度
        int len_p = p.length(); // p的长度
        sort(p.begin(), p.end()); // 对p进行排序

        int i = 0, j = i + len_p - 1; // 滑动窗口
        while (j < len_s) {
            // 将滑动窗口中的字串取出，存放到tmp中
            string tmp = "";
            
            tmp.append(s.substr(i, len_p));
            
            sort(tmp.begin(), tmp.end());
            if (!tmp.compare(p)) { // 若两个字符串相等
                ans.push_back(i);
            }

            i++, j++; // 往后移动滑动窗口
        }
        return ans;
    }
};
```



### 正确做法

同样的滑动窗口思路，但是判断字符串是否是字母异位词，改用长度为26的数组来记录遍历到的字串中字母出现的次数与 p 字符串中字母出现的次数，若两者相同，则是字母异位词。代码如下

```c++
class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        int len_s = s.length(); // s的长度
        int len_p = p.length(); // p的长度
        if (len_s < len_p) return vector<int>();

        vector<int> ans;
        vector<int> sCNT(26); // 记录s串中字母出现次数
        vector<int> pCNT(26); // 记录p串中字母出现次数
        for (int i = 0; i < len_p; ++i) {
            ++sCNT[s[i] - 'a'];
            ++pCNT[p[i] - 'a'];
        }

        // 0位置开始的滑动窗口中的字符串与目标字符串互为字母异位
        if (sCNT == pCNT) ans.push_back(0); 

        for (int i = 0; i < len_s - len_p; ++i) {
            // 移动窗口
            --sCNT[s[i] - 'a'];
            ++sCNT[s[i + len_p] - 'a'];

            if (sCNT == pCNT) {
                ans.push_back(i + 1);
            }
        }
        return ans;
    }
};
```



## 10.和为K的子数组（前缀和+哈希）

给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 *该数组中和为 `k` 的子数组的个数* 。

**子数组是数组中元素的连续非空序列**。

**示例 1：**

```
输入：nums = [1,1,1], k = 2
输出：2
```

**示例 2：**

```
输入：nums = [1,2,3], k = 3
输出：2
```

**提示：**

- `1 <= nums.length <= 2 * 10^4`
- `-1000 <= nums[i] <= 1000`
- `-10^7 <= k <= 10^7`



### 思路

**前缀和+哈希表**

由于枚举肯定会超时，所以要对其进行优化。我们可以通过前缀和尝试减少时间复杂度，定义 pre[i] 表示 nums 数组中前 i 个元素的和，那么 pre[i] = pre[i - 1] + nums[i]。

于是**“子数组[j...i]的和为k”**这个条件可以转化为**“pre[i] - pre[j - 1] == k”**。经过简单移项后可得符合条件的下标 j 需要满足条件：**pre[j - 1] == pre[i] - k**。所以考虑以 i 结尾的和为 k 的子数组个数时，只需要统计有多少个前缀和为**pre[i] - k** 的 **pre[j]** 即可。因为 j 在 i 之前，第一想法就是每次移动 i 时，需要从头往 i 遍历一次来看有没有满足条件的 pre[j]，但这会增加复杂度，所以我们可以通过哈希表来进行优化，**以和为键，该和出现的次数为值**，最后查找的时候就能在O(1)的时间内得到答案。代码如下

```c++
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<int, int> mp;
        mp[0] = 1; // 和为0出现了1次
        int ans = 0, pre = 0;
        for (auto& x : nums) { // 遍历数组（移动i）
            pre += x; // 记录前缀和
            if (mp.find(pre - k) != mp.end()) {
                ans += mp[pre - k]; // 找到了一组[j, i]子数组
            }
            mp[pre]++; // 当前前缀和出现次数+1
        }
        return ans;
    }
};
```



## 11.滑动窗口最大值（优先队列）

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。

**示例 1：**

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**示例 2：**

```
输入：nums = [1], k = 1
输出：[1]
```

**提示：**

- `1 <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`
- `1 <= k <= nums.length`



### 思路

与之前几题类似，不能每次移动滑动窗口时就从窗口第一个元素开始枚举，这样一定会超时。

考虑到这个题滑动窗口的操作过程，很容易联想到队列，再根据他要返回滑动窗口中的最大值，可以考虑使用**优先队列**。

对于优先队列，**其中的元素默认是从大到小排序**，所以我们可以先将数组中前 k 个元素压入优先队列中，记录一次队顶元素（当前滑动窗口的最大值）。之后从数组的第 **(k + 1)** 个元素开始，每次将一个元素压入队列，此时的滑动窗口超出了其承载量（多了一个元素），于是我们可以对队顶元素进行判断，看他是否在当前滑动窗口中**(根据下标 index 是否在滑动窗口中来判断)**。考虑到这种做法，我们需要用**单独的一个整型变量来记录滑动窗口的下标，并且还要记录每个元素在原数组中的下标 index**，对于后者，我们可以用 **pair 二元组**来实现。具体代码如下

```c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int len = nums.size();
        // 优先队列默认从大到小排序
        // 在pair中存放的是“值-index”的二元组
        priority_queue<pair<int, int>> q;
        
        // 先将前k个元素压入优先队列中
        for (int i = 0; i < k; ++i) {
            q.emplace(nums[i], i);
        }

        vector<int> ans = {q.top().first}; // 先存放当前队列的最大值
        for (int i = k; i < len; i++) {
            q.emplace(nums[i], i); // 向队列中添加新元素
            // 若当前最大值不在滑动窗口中，则弹出队列
            while (q.top().second <= i - k) {
                q.pop();
            }
            ans.push_back(q.top().first); // 向答案中添加当前窗口中的最大值
        }
        return ans;
    }
};
```





## 12.最大子数组和

给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组**是数组中的一个连续部分。



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

简单动态规划思路，原问题要求最大和的连续子数组，而对于每个连续子数组，都**唯一对应着一个末尾元素**。在数组 nums 中，每个元素都可以作为某个子数组的末尾元素。所以我们只用求出以 nums[i] 结尾的**最大连续子数组**的最大和即可。

定义一个 dp[len] 数组，len 是数组的长度，dp[i] 表示以 nums[i] 结尾的最大子数组和。遍历一次 nums 数组，对于每个元素，判断 (dp[i - 1] + nums[i]) 和 nums[i] 哪个更大，将更大的值赋给dp[i]，并实时更新max的值即可。代码如下

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int len = nums.size();
        int dp[len + 5]; // dp[i]表示以nums[i]结尾的最大子数组和

        dp[0] = nums[0];
        int max = dp[0];
        for (int i = 1; i < len; ++i) {
            if (nums[i] + dp[i - 1] > nums[i]) 
                dp[i] = nums[i] + dp[i - 1];
            else dp[i] = nums[i];
            if (dp[i] > max) max = dp[i];
        }
        return max;
    }
};
```





## 13.合并区间

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 *一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间* 。



**示例 1：**

```
输入：intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
```

**示例 2：**

```
输入：intervals = [[1,4],[4,5]]
输出：[[1,5]]
解释：区间 [1,4] 和 [4,5] 可被视为重叠区间。
```



**提示：**

- `1 <= intervals.length <= 10^4`
- `intervals[i].length == 2`
- `0 <= starti <= endi <= 10^4`



### 思路

对于题目给定的这样一个数组，我们可以先按照左边界进行升序排序。这样操作后，可以合并的区间一定在原来的数组中位置连续。

![image-20240509203403984.png](https://s2.loli.net/2024/08/05/a52zfLYkQx1uTE9.png)

然后定义一个ans数组存放结果，将ans数组中最后一个元素的右边界（整个ans的右边界）与遍历到的区间的左边界进行比较，选择更新ans的右边界或者将该区间压入ans。具体代码如下

```c++
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if (intervals.size() == 0) return {};
        // 先按照左边界排序
        sort(intervals.begin(), intervals.end());
        vector<vector<int>> merged;
        for (int i = 0; i < intervals.size(); ++i) {
            int left = intervals[i][0], right = intervals[i][1];
            // 如果merged数组为空，或者与merged的最后一个数组没有重叠部分
            if (!merged.size() || merged.back()[1] < left) {
                merged.push_back({left, right});
            } else { // 有重叠部分，更新merged末尾的右边界（合并）
                merged.back()[1] = max(merged.back()[1], right);
            }
        }
        return merged;
    }
};
```





## 14.最小覆盖子串

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。



**注意：**

- 对于 `t` 中重复字符，我们寻找的子字符串中该字符数量必须不少于 `t` 中该字符数量。
- 如果 `s` 中存在这样的子串，我们保证它是唯一的答案。



**示例 1：**

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
解释：最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
```

**示例 2：**

```
输入：s = "a", t = "a"
输出："a"
解释：整个字符串 s 是最小覆盖子串。
```

**示例 3:**

```
输入: s = "a", t = "aa"
输出: ""
解释: t 中两个字符 'a' 均应包含在 s 的子串中，
因此没有符合条件的子字符串，返回空字符串。
```



**提示：**

- `m == s.length`
- `n == t.length`
- `1 <= m, n <= 10^5`
- `s` 和 `t` 由英文字母组成



### 思路

考虑到这是一个查找最小子串问题，我们可以往子集问题上联想，自然而然就会考虑使用哈希表来尝试解题。该问题的一个核心点是**我们如何判断当前在字符串s 中的子串包含了字符串 t**。对于找字符串 s 中的子串，我们可以用滑动窗口。我们再考虑哈希表，我们创建一个哈希表 hash_t 来存放字符串 t 中出现的字母与其出现的次数（**这样定义键值对是考虑了字符串 t 中有重复字符**），然后我们再创建另一个哈希表 hash_s 来维护当前字符串 s 中遍历到的子串中的字母及其出现次数。**对于拿到的子串，这个子串对应的哈希表中要包含 t 的哈希表中的所有字符，并且对应的个数都不小于 t 的哈希表中各个字符的个数**。

当我们拿到包含 t 的子串时，再考虑收缩滑动窗口，得到一个最小的子串。

具体代码如下

```c++
class Solution {
public:
    // ori记录t字符串，cnt记录s字符串
    unordered_map <char, int> ori, cnt; 

    // 判断是否包含子串
    bool check() {
        for (const auto &p: ori) {
            if (cnt[p.first] < p.second) {
                return false;
            }
        }
        return true;
    }

    string minWindow(string s, string t) {
        // 维护一个t串的哈希表
        for (const auto &c: t) {
            ++ori[c];
        }

        // 左右指针
        int l = 0, r = -1;
        // len为当前s中的长度，ansL与ansR记录答案子串的首尾索引
        int len = INT_MAX, ansL = -1, ansR = -1;

        while (r < int(s.size())) {
            // 扩展窗口
            if (ori.find(s[++r]) != ori.end()) {
                ++cnt[s[r]];
            }
            // 判断子串是否包含t字符串
            while (check() && l <= r) {
                // 长度更短则更新
                if (r - l + 1 < len) {
                    len = r - l + 1;
                    ansL = l; // 更新答案首部索引
                }
                // 尝试收缩滑动窗口
                if (ori.find(s[l]) != ori.end()) {
                    --cnt[s[l]];
                }
                ++l;
            }
        }
        return ansL == -1 ? "" : s.substr(ansL, len);
    }
};
```





## 15.轮转数组

给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。



**示例 1:**

```
输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
解释:
向右轮转 1 步: [7,1,2,3,4,5,6]
向右轮转 2 步: [6,7,1,2,3,4,5]
向右轮转 3 步: [5,6,7,1,2,3,4]
```

**示例 2:**

```
输入：nums = [-1,-100,3,99], k = 2
输出：[3,99,-1,-100]
解释: 
向右轮转 1 步: [99,-1,-100,3]
向右轮转 2 步: [3,99,-1,-100]
```



**提示：**

- `1 <= nums.length <= 10^5`
- `-2^31 <= nums[i] <= 2^31 - 1`
- `0 <= k <= 10^5`



### 思路

直接模拟，最后一个测试点数据太大会爆超时

```c++
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        while (k--) {
            for (int i = n - 1; i > 0; --i) {
                swap(nums[i], nums[i-1]);
            }
        }
    }
};
```

考虑直接**将末尾的 k 个元素扔到数组前面**（通过新创建一个数组实现），就能得到答案。但这样的做法有一个前提：数组长度 len 要比 k 更大。所以可以分两种情况解答

- len > k 时采用将末尾的 k 个元素扔到数组前面
- len <= k 时直接模拟

代码如下

```c++
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        if (k < n) {
            vector<int> res;
            for (int i = n - k; i < n; ++i) {
                res.push_back(nums[i]);
            }
            for (int i = 0; i < n - k; ++i) {
                res.push_back(nums[i]);
            }
            nums = res;
        } else {
            while (k--) {
                for (int i = n - 1; i > 0; --i) {
                    swap(nums[i], nums[i-1]);
                }
            }
        }
    }
};
```

为了代码更简短，可以将两种情况进行合并，我们只需要遍历原数组，将原数组下标为 i 的元素放到新数组下标为 **(i + k) mod n** 的位置

```c++
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> newArr(n);
        for (int i = 0; i < n; ++i) {
            newArr[(i + k) % n] = nums[i];
        }
        nums.assign(newArr.begin(), newArr.end());
    }
};
```





## 16.除自身以外数组的乘积

给你一个整数数组 `nums`，返回 *数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积* 

题目数据 **保证** 数组 `nums`之中任意元素的全部前缀元素和后缀的乘积都在 **32 位** 整数范围内

请 **不要使用除法，**且在 `O(n)` 时间复杂度内完成此题



**示例 1:**

```
输入: nums = [1,2,3,4]
输出: [24,12,8,6]
```

**示例 2:**

```
输入: nums = [-1,1,0,-3,3]
输出: [0,0,9,0,0]
```



**提示：**

- `2 <= nums.length <= 10^5`
- `-30 <= nums[i] <= 30`
- **保证** 数组 `nums`之中任意元素的全部前缀元素和后缀的乘积都在 **32 位** 整数范围内



### 思路

暴力的思路就是遍历一边 nums 数组，每对于遍历到的每个元素分别计算除它以外的乘积。但看数据范围，一定会超时。考虑问题的性质，这个问题可以通过**前缀和**的思想解决。

对于 nums 数组中的每个元素，我们可以通过下面的方法计算**除它以外的乘积**：计算结果分为两部分，以当前元素为界，该元素前半部分与该元素后半部分。在遍历 nums 数组前，我们先行分别计算这两部分的结果存放到数组中，然后再遍历 nums 数组，对于每个取到的元素，分别到前面创建的两个数组里取值相乘即可。代码如下

```c++
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        vector<int> ans;
        vector<int> pre_front; // 前缀乘积
        vector<int> pre_back; // 后缀乘积
        int len = nums.size();

        int tmp = 1;
        // 计算前缀乘积
        for (int i = 0; i < len; ++i) {
            tmp *= nums[i];
            pre_front.push_back(tmp);
        }
        tmp = 1;
        // 计算后缀乘积
        for (int i = len - 1; i >= 0; --i) {
            tmp *= nums[i];
            pre_back.push_back(tmp);
        }

        for (int i = 0; i < len; ++i) {
            if (i == 0) {
                ans.push_back(pre_back[len - 2]);
            } else if (i == len - 1) {
                ans.push_back(pre_front[len - 2]);
            } else {
                ans.push_back(pre_front[i - 1] * pre_back[len - 2 - i]);
            }
        }
        return ans;
    }
};
```





## 17.缺失的第一个正数

给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数。

请你实现时间复杂度为 `O(n)` 并且只使用常数级别额外空间的解决方案。



**示例 1：**

```
输入：nums = [1,2,0]
输出：3
解释：范围 [1,2] 中的数字都在数组中。
```

**示例 2：**

```
输入：nums = [3,4,-1,1]
输出：2
解释：1 在数组中，但 2 没有。
```

**示例 3：**

```
输入：nums = [7,8,9,11,12]
输出：1
解释：最小的正数 1 没有出现。
```



**提示：**

- `1 <= nums.length <= 10^5`
- `-2^31 <= nums[i] <= 2^31 - 1`



### 思路

先不考虑如何优化，我们要找出没有出现的最小正数，先对给定的数组排序，然后遍历该数组，找到第一个正数。因为事先已经排好序，所以这个一定是最小正数。**如果这个正数不是1，那么答案就直接返回1**，否则就继续判断。

我们从最小正数的下一位开始，**找到两个相邻且两者之差不为1或0的数 nums[i]和nums[i-1]**，返回 **nums[i-1] + 1** 即为答案。如果一直找到末尾都没有找到这样的两个数，**那么就说明数组中的正整数都连续**，返回最后一个元素 + 1 即可。

代码如下

```c++
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int len = nums.size();
        long long ans;
        sort(nums.begin(), nums.end());
        // 假设给定数组的正整数部分都连续，答案就是最后一个元素+1
        if (nums[len - 1] > 0) {
            // 有一个数据点是INT_MAX，这里先强转一下防溢出
            ans = (long long)nums[len - 1] + 1;
        } else { // 若数组中没有正整数，答案就为1
            ans = 1;
        }
        int min = 1;
        int flag; // 记录最小正整数的index
        for (int i = 0; i < len; ++i) {
            if (nums[i] > 0) {
                min = nums[i];
                flag = i;
                break;
            }
        }

        if (min != 1) {
            return 1;
        } else { // 寻找不连续的两个正整数
            for (int i = flag + 1; i < len; ++i) {
                if (nums[i] - nums[i - 1] != 1 && nums[i] - nums[i - 1] != 0) {
                    ans = nums[i - 1] + 1;
                    break;
                }
            }
        }
        return ans;
    }
};
```



**正解**

上述排序做法的时间复杂度明显不为 `O(n)` ，我们要找到一个时间复杂度为 `O(n)` 的做法，可以借助哈希表。我们将 `nums` 的所有元素都加入到哈希表中，并且记录 `nums` 的最大值 `mx` 。然后我们从 1 开始枚举正整数，枚举的同时判断当前正整数是否在哈希表中，如果不在就是我们要找的答案。

```c++
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        unordered_set<int> st(nums.begin(), nums.end());
        int mx = *max_element(nums.begin(), nums.end());
        for (int i = 1; i <= mx; ++i) {
            if (!st.contains(i)) {
                return i;
            }
        }
        return mx > 0 ? mx + 1 : 1;
    }
};
```





## 18.矩阵置零

给定一个 `m x n` 的矩阵，如果一个元素为 **0** ，则将其所在行和列的所有元素都设为 **0** 。请使用 **[原地](http://baike.baidu.com/item/原地算法)** 算法。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/08/17/mat1.jpg)

```
输入：matrix = [[1,1,1],[1,0,1],[1,1,1]]
输出：[[1,0,1],[0,0,0],[1,0,1]]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/08/17/mat2.jpg)

```
输入：matrix = [[0,1,2,0],[3,4,5,2],[1,3,1,5]]
输出：[[0,0,0,0],[0,4,5,0],[0,3,1,0]]
```



**提示：**

- `m == matrix.length`
- `n == matrix[0].length`
- `1 <= m, n <= 200`
- `-2^31 <= matrix[i][j] <= 2^31 - 1`



### 思路

最直观的方案是使用额外的O(mn)的空间，记录原矩阵中哪些是0，将其所在的行列均置为0即可。代码如下

```c++
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int m = matrix.size(); // 行数
        int n = matrix[0].size(); // 列数
        bool isZero[205][205] = {false}; // 判断原来矩阵中哪些是0

        for (int i = 0; i < m; ++i) 
            for (int j = 0; j < n; ++j) 
                if (matrix[i][j] == 0) isZero[i][j] = true;

        for (int i = 0; i < m; ++i) 
            for (int j = 0; j < n; ++j)
                if (isZero[i][j]) {
                    for (int p = 0; p < m; ++p) 
                        matrix[p][j] = 0;
                    for (int q = 0; q < n; ++q) 
                        matrix[i][q] = 0;
                }
    }
};
```



对于记录0的位置，可以进一步减小内存到O(m+n)。我们并不需要用整个矩阵存储为0的坐标，只需要用vector记录为0的坐标即可。代码如下

```c++
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int m = matrix.size(), n = matrix[0].size();
        vector<pair<int, int>> zero;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (matrix[i][j] == 0) {
                    zero.push_back({i, j});
                }
            }
        }
        for (auto& [x, y]: zero) {
            for (int i = 0; i < m; ++i) matrix[i][y] = 0;
            for (int j = 0; j < n; ++j) matrix[x][j] = 0;
        }
    }
};
```



还可以进一步将空间复杂度降为O(1)。我们可以用矩阵的第一行和第一列代替标记数组，但这样会导致原数组的第一行和第一列被修改，无法记录它们原来是否包含0。**所以我们需要额外使用两个标记变量分别记录第一行和第一列是否包含0**。

实际操作起来，我们先预处理两个标记变量，接着使用其他行和列去处理第一行和第一列，再反过来根据第一行和第一列更新剩余矩阵，最后用两个标记变量更新第一行与第一列即可。

```c++
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int m = matrix.size();
        int n = matrix[0].size();
        int flag_col0 = false, flag_row0 = false;
        for (int i = 0; i < m; i++) {
            if (!matrix[i][0]) {
                flag_col0 = true;
            }
        }
        for (int j = 0; j < n; j++) {
            if (!matrix[0][j]) {
                flag_row0 = true;
            }
        }
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                if (!matrix[i][j]) {
                    matrix[i][0] = matrix[0][j] = 0;
                }
            }
        }
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                if (!matrix[i][0] || !matrix[0][j]) {
                    matrix[i][j] = 0;
                }
            }
        }
        if (flag_col0) {
            for (int i = 0; i < m; i++) {
                matrix[i][0] = 0;
            }
        }
        if (flag_row0) {
            for (int j = 0; j < n; j++) {
                matrix[0][j] = 0;
            }
        }
    }
};
```





## 19.螺旋矩阵

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/13/spiral1.jpg)

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[1,2,3,6,9,8,7,4,5]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/11/13/spiral.jpg)

```
输入：matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
输出：[1,2,3,4,8,12,11,10,9,5,6,7]
```

 

**提示：**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 10`
- `-100 <= matrix[i][j] <= 100`



### 思路

直接模拟，使用额外数组模拟方向变化

```c++
#include<bits/stdc++.h>
using namespace std;

vector<int> solve(vector<vector<int>> matrix) {
	vector<vector<int>> DIRS;
    DIRS.push_back({0, 1});
    DIRS.push_back({1, 0});
    DIRS.push_back({0, -1});
    DIRS.push_back({-1, 0});
    int m = matrix.size(), n = matrix[0].size();
    vector<vector<bool>> vis(m, vector<bool>(n, false));
    int i = 0, j = 0, dir = 0;
    vector<int> res;
    // 数字个数
    int cnt = m * n;
    while (cnt--) {
        res.push_back(matrix[i][j]);
        vis[i][j] = true;
        int x = i + DIRS[dir][0];
        int y = j + DIRS[dir][1];
        // 如果该方向下一个点出界或者已经访问过，那么就变换方向
        if (x < 0 || x >= m || y < 0 || y >= n || vis[x][y]) {
            dir = (dir + 1) % 4;
        }
        i += DIRS[dir][0];
        j += DIRS[dir][1];
    }
    return res;
}

void print(vector<int> res) {
    for (auto x : res) cout << x << " ";
    cout << endl;
}

int main() {
    // test cases
    return 0;
}
```





## 20.旋转图像

给定一个 *n* × *n* 的二维矩阵 `matrix` 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在**[ 原地](https://baike.baidu.com/item/原地算法)** 旋转图像，这意味着你需要直接修改输入的二维矩阵。**请不要** 使用另一个矩阵来旋转图像。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/08/28/mat1.jpg)

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[[7,4,1],[8,5,2],[9,6,3]]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/08/28/mat2.jpg)

```
输入：matrix = [[5,1,9,11],[2,4,8,10],[13,3,6,7],[15,14,12,16]]
输出：[[15,13,2,5],[14,3,4,1],[12,6,8,9],[16,7,10,11]]
```

 

**提示：**

- `n == matrix.length == matrix[i].length`
- `1 <= n <= 20`
- `-1000 <= matrix[i][j] <= 1000`



### 思路

经过观察，旋转后的矩阵的每一行的内容，是从**原来矩阵每一列的最后一个元素开始一直到第一个元素为止**，所以很容易能想到先创建一个新矩阵，再用这种方法遍历原矩阵，将旋转后的结果存储到新矩阵中，最后用新矩阵将原矩阵覆盖掉即可。代码如下

```c++
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        vector<vector<int>> tmp(n);
        for (int i = 0; i < n; ++i) {
            for (int j = n - 1; j >= 0; --j) {
                tmp[i].push_back(matrix[j][i]);
            }
        }
        matrix = tmp;
    }
};
```



但题目要求不能新建矩阵，所以下面换一个思路：**通过翻转矩阵实现**。

注意到目标矩阵可以通过原矩阵进行翻转变换得到，具**体翻转过程是先通过水平轴翻转一次，再通过主对角线翻转一次**。

水平轴翻转

![1.png](https://s2.loli.net/2024/08/05/2kXIbcq1pMmCEJO.png)



主对角线翻转

![1.png](https://s2.loli.net/2024/08/05/PwoedBiglJcqvD3.png)



水平轴翻转的坐标变换如下

![1.png](https://s2.loli.net/2024/08/05/XiQcLsW1F7R8eAG.png)

主对角线翻转变换的坐标变换如下

![1.png](https://s2.loli.net/2024/08/05/FmvGCubWYI85fHT.png)



具体代码如下

```c++
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        // 水平翻转
        for (int i = 0; i < n / 2; ++i) {
            for (int j = 0; j < n; ++j) {
                swap(matrix[i][j], matrix[n - i - 1][j]);
            }
        }
        // 主对角线翻转
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < i; ++j) {
                swap(matrix[i][j], matrix[j][i]);
            }
        }
    }
};
```







## 21.搜索二维矩阵Ⅱ

编写一个高效的算法来搜索 `*m* x *n*` 矩阵 `matrix` 中的一个目标值 `target` 。该矩阵具有以下特性：

- 每行的元素从左到右升序排列。
- 每列的元素从上到下升序排列。

 

**示例 1：**

![image.png](https://s2.loli.net/2025/02/25/GCuRgvO8i9co1DJ.png)

```
输入：matrix = [[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]], target = 5
输出：true
```

**示例 2：**

![image.png](https://s2.loli.net/2025/02/25/xluEsgtJamc8nLv.png)

```
输入：matrix = [[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]], target = 20
输出：false
```

 

**提示：**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= n, m <= 300`
- `-10^9 <= matrix[i][j] <= 10^9`
- 每行的所有元素从左到右升序排列
- 每列的所有元素从上到下升序排列
- `-10^9 <= target <= 10^9`



### 思路

一次剪枝搜索就能过，代码如下

```c++
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int m = matrix.size();
        int n = matrix[0].size();

        for (int i = 0; i < m; ++i) {
            // 剪枝
            if (target > matrix[i][n - 1]) continue;
            for (int j = 0; j < n; ++j) {
                if (target == matrix[i][j]) return true;
            }
        }
        return false;
    }
};
```

但只进行这一次剪枝效率还是比较低，接下来考虑一个更高效的搜索方法

我们从每一行的最后一个元素开始搜索，如果当前元素小于 target ，就直接跳到下一行，列逐渐减小即可。代码如下

```c++
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int m = matrix.size();
        int n = matrix[0].size();
        int j = n - 1;

        for (int i = 0; i < m; ++i) {
            if (target > matrix[i][j]) continue;
            while (target < matrix[i][j] && j > 0) j--;
            if (matrix[i][j] == target) return true;
        }
        return false;
    }
};
```







## 22.相交链表

给你两个单链表的头节点 `headA` 和 `headB` ，请你找出并返回两个单链表相交的起始节点。如果两个链表不存在相交节点，返回 `null` 。

图示两个链表在节点 `c1` 开始相交**：**

[![image.png](https://s2.loli.net/2025/02/18/o2JH3umn5WISqwf.png)](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)

题目数据 **保证** 整个链式结构中不存在环。

**注意**，函数返回结果后，链表必须 **保持其原始结构** 。

**自定义评测：**

**评测系统** 的输入如下（你设计的程序 **不适用** 此输入）：

- `intersectVal` - 相交的起始节点的值。如果不存在相交节点，这一值为 `0`
- `listA` - 第一个链表
- `listB` - 第二个链表
- `skipA` - 在 `listA` 中（从头节点开始）跳到交叉节点的节点数
- `skipB` - 在 `listB` 中（从头节点开始）跳到交叉节点的节点数

评测系统将根据这些输入创建链式数据结构，并将两个头节点 `headA` 和 `headB` 传递给你的程序。如果程序能够正确返回相交节点，那么你的解决方案将被 **视作正确答案** 。

 

**示例 1：**

[![image.png](https://s2.loli.net/2025/02/18/Lsg5Hva3oxV4mpQ.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_1.png)

```
输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,6,1,8,4,5], skipA = 2, skipB = 3
输出：Intersected at '8'
解释：相交节点的值为 8 （注意，如果两个链表相交则不能为 0）。
从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,6,1,8,4,5]。
在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。
— 请注意相交节点的值不为 1，因为在链表 A 和链表 B 之中值为 1 的节点 (A 中第二个节点和 B 中第三个节点) 是不同的节点。换句话说，它们在内存中指向两个不同的位置，而链表 A 和链表 B 中值为 8 的节点 (A 中第三个节点，B 中第四个节点) 在内存中指向相同的位置。
```

 

**示例 2：**

[![image.png](https://s2.loli.net/2025/02/18/smjODC94YBeIlnf.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_2.png)

```
输入：intersectVal = 2, listA = [1,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
输出：Intersected at '2'
解释：相交节点的值为 2 （注意，如果两个链表相交则不能为 0）。
从各自的表头开始算起，链表 A 为 [1,9,1,2,4]，链表 B 为 [3,2,4]。
在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。
```

**示例 3：**

[![image.png](https://s2.loli.net/2025/02/18/O7zHAZreYUS5nbQ.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_3.png)

```
输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
输出：null
解释：从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。
由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
这两个链表不相交，因此返回 null 。
```

 

**提示：**

- `listA` 中节点数目为 `m`
- `listB` 中节点数目为 `n`
- `1 <= m, n <= 3 * 10^4`
- `1 <= Node.val <= 10^5`
- `0 <= skipA <= m`
- `0 <= skipB <= n`
- 如果 `listA` 和 `listB` 没有交点，`intersectVal` 为 `0`
- 如果 `listA` 和 `listB` 有交点，`intersectVal == listA[skipA] == listB[skipB]`

 

### 思路

本题可采用**双指针**方法。首先判断两个链表是否为空，只有两个链表都非空，才可能相交。

当两个链表均不为空，我们创建两个指针 pa 和 pb ，初始化分别指向链表A和链表B的头节点，然后将两个指针同步往后挪。**当两个指针指向了同一个节点（pa == pb）**，我们再判断两个指针是否为空，如果为空就返回NULL，否则就找到了答案。倘若一个指针已经走到了头，还没有找到同一个节点，就将该指针指向另一条链表的头节点继续遍历。

下面验证这种方法的正确性：**考虑两种情况，第一种情况是两个链表相交，第二种情况是两个链表不相交。**

**情况一：两个链表相交**

**链表 headA** 和**headB**的长度分别为 **m** 和 **n**。假设**链表 headA** 的不相交部分有 **a** 个节点，**headB**的不相交部分有 **b** 个节点，两个链表相交的部分有 **c** 个节点，则有 **a + c = m，b + c = n**。

- 如果 **a = b**，则两个指针会同时到达两个链表相交的节点，此时返回相交的节点；
- 如果 **a ≠ b**，则**指针 pa** 会遍历完 **headA**，指针 **pb **会遍历完 **headB**，两个指针不会同时到达链表的尾节点，然后**指针 pa** 移到 **headB** 的头节点，**指针 pb** 移到 headA 的头节点，然后两个指针继续移动，在 **pa** 移动了 **a + c + b** 次，**pb** 移动了 **b + c + a** 次后，两个指针会同时到达两个链表相交的节点，该节点也是两个指针第一次同时指向的节点，此时返回相交的节点



情况二：两个链表不相交

**链表 headA** 和 **headB** 的长度分别是 **m** 和 **n**。考虑当 **m=n** 和 **m≠n** 时，两个指针分别会如何移动：

- 如果 **m=n**，则两个指针会同时到达两个链表的尾节点，然后同时变成空值 **null**，此时返回 **null**；
- 如果 **m≠n**，则由于两个链表没有公共节点，两个指针也不会同时到达两个链表的尾节点，因此两个指针都会遍历完两个链表，在指针 pA 移动了 **m+n** 次、指针 pB 移动了 **n+m** 次之后，两个指针会同时变成空值 **null**，此时返回 **null**。



代码如下

```c++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if (headA == NULL || headB == NULL) return NULL;
        ListNode *pa = headA, *pb = headB;
        while (pa != pb) {
            pa = pa == NULL ? headB : pa->next;
            pb = pb == NULL ? headA : pb->next;
        }
        return pa;
    }
};
```





## 23.反转链表

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/02/19/rev1ex1.jpg)

```
输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/02/19/rev1ex2.jpg)

```
输入：head = [1,2]
输出：[2,1]
```

**示例 3：**

```
输入：head = []
输出：[]
```

 

**提示：**

- 链表中节点的数目范围是 `[0, 5000]`
- `-5000 <= Node.val <= 5000`



### 思路

**搞清楚对于每个节点要进行哪些操作，以及这些操作的顺序就能做出来**

在反转链表的过程中，我们记**指向当前节点的前一个节点的指针为 prev，指向下一个节点的指针为 next，指向当前节点的指针为curr**。对于单个节点的处理可以分为以下步骤**（注意区分 curr->next，和 next）**

1. **将 curr->next 指向 prev**
2. **将 prev 指向 curr**
3. **将 curr 指向 next**

代码如下

```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode *prev = nullptr;
        ListNode *curr = head;
        while (curr) {
            ListNode *next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
};
```

除了以上思路，我们还可以用栈来实现链表的反转：

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        stack<int> st;
        ListNode* p = head;
        while (p != nullptr) {
            st.push(p->val);
            p = p->next;
        }
        ListNode* dummy = new ListNode();
        p = dummy;
        while (!st.empty()) {
            ListNode *temp = new ListNode(st.top());
            p->next = temp;
            p = p->next;
            st.pop();
        }
        return dummy->next;
    }
};
```



## 24.回文链表

给你一个单链表的头节点 `head` ，请你判断该链表是否为回文链表。如果是，返回 `true` ；否则，返回 `false` 。



 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/03/03/pal1linked-list.jpg)

```
输入：head = [1,2,2,1]
输出：true
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/03/03/pal2linked-list.jpg)

```
输入：head = [1,2]
输出：false
```

 

**提示：**

- 链表中节点数目在范围`[1, 10^5]` 内
- `0 <= Node.val <= 9`



### 思路

遍历链表，将值存放到数组里，那么链表的回文判断就变成了线性表的回文判断。代码如下

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        vector<int> f;
        ListNode* p = head;
        while (p) {
            f.push_back(p->val);
            p = p->next;
        }
        int n = f.size();
        for (int i = 0; i < n / 2; ++i) {
            if (f[i] != f[n-1-i]) {
                return false;
            }
        }
        return true;
    }
};
```

这种做法的空间复杂度为O(n)，可以进一步减小空间复杂度，其实思路也很简单：找到链表的中间结点 mid，然后反转从 mid 到链表末尾这段，反转第二段得到头节点 head2 ，最后同时遍历 head 和 head2 两个链表，直到 head2 链表遍历结束，判断有无节点值对应不相等。

注意：找中间结点使用快慢指针：

- 如果链表节点数为奇数，那么找的就是正中间的节点

![image.png](https://s2.loli.net/2025/03/04/ahSUk1yvD7LmsjC.png)

- 如果链表节点数为偶数，那么找的是正中间右边的节点

![image.png](https://s2.loli.net/2025/03/04/TA6C1xokEc9URNn.png)



**Code**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
    // 找链表的中间结点
    ListNode* midNode(ListNode* head) {
        ListNode* slow = head, *fast = head;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow;
    }

    // 反转链表
    ListNode* reverseList(ListNode* head) {
        ListNode* pre = nullptr, *cur = head;
        while (cur) {
            ListNode* nxt = cur->next;
            cur->next = pre;
            pre = cur;
            cur = nxt;
        }
        return pre;
    }

public:
    bool isPalindrome(ListNode* head) {
        ListNode* mid = midNode(head);
        ListNode* head2 = reverseList(mid);
        while (head2) {
            if (head->val != head2->val) {
                return false;
            }
            head = head->next;
            head2 = head2->next;
        }    
        return true;
    }
};
```





## 25.环形链表

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。

*如果链表中存在环* ，则返回 `true` 。 否则，返回 `false` 。

 

**示例 1：**

![image.png](https://s2.loli.net/2025/02/18/nRFy9ZtdaK8fHNv.png)

```
输入：head = [3,2,0,-4], pos = 1
输出：true
解释：链表中有一个环，其尾部连接到第二个节点。
```

**示例 2：**

![image.png](https://s2.loli.net/2025/02/18/CrTm6S8bGx5ilIN.png)

```
输入：head = [1,2], pos = 0
输出：true
解释：链表中有一个环，其尾部连接到第一个节点。
```

**示例 3：**

![image.png](https://s2.loli.net/2025/02/18/bqDX9FMQWHNaEpf.png)

```
输入：head = [1], pos = -1
输出：false
解释：链表中没有环。
```

 

**提示：**

- 链表中节点的数目范围是 `[0, 10^4]`
- `-10^5 <= Node.val <= 10^5`
- `pos` 为 `-1` 或者链表中的一个 **有效索引** 。



### 思路

比较直接的想法就是创建一个哈希集合，然后遍历一遍链表，遍历的过程中判断这个节点是否在哈希集合里，如果不在就加入到哈希集合，否则就说明出现了环，直接返回true，代码如下

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    bool hasCycle(ListNode *head) {
        unordered_set<ListNode*> st;
        ListNode* p = head;
        while (p) {
            if (!st.count(p)) st.insert(p);
            else return true;
            p = p->next;
        }
        return false;
    }
};
```

以上做法使用了额外的空间来记录之前的节点是否出现过，以这个为依据来判断环的存在。

其实可以用另一种方法省去这个空间的开支，下面介绍**「Floyd 判圈算法」（又称龟兔赛跑算法）**

**假想「乌龟」和「兔子」在链表上移动，「兔子」跑得快，「乌龟」跑得慢。当「乌龟」和「兔子」从链表上的同一个节点开始移动时，如果该链表中没有环，那么「兔子」将一直处于「乌龟」的前方；如果该链表中有环，那么「兔子」会先于「乌龟」进入环，并且一直在环内移动。等到「乌龟」进入环时，由于「兔子」的速度快，它一定会在某个时刻与乌龟相遇，即套了「乌龟」若干圈。**

![1.png](https://s2.loli.net/2024/08/05/6rtx1KJoEPvFTCX.png)

从而我们可以用快慢指针来模拟兔子和乌龟，但有以下几点细节需要注意

我们要规定初始时慢指针在位置head，快指针在位置head->next。**这是因为我们使用的是while循环，循环条件先于循环体。由于循环条件一定是判断快慢指针是否重合，如果我们将两个指针初始都置于head，那么while循环就不会执行**。**或者我们可以用do-while循环，这样两个指针就可以设置在同一个起点了**，代码如下

```c++
class Solution {
public:
    bool hasCycle(ListNode *head) {
        if (head == nullptr || head->next == nullptr) return false;
        // 快慢指针
        ListNode *fast, *slow;
        fast = slow = head;
        do {
            if (fast == nullptr || fast->next == nullptr) return false;
            fast = fast->next->next;
            slow = slow->next;
        } while (fast != slow);
        return true;
    }
};
```





## 26.环形链表Ⅱ

给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 *如果链表无环，则返回 `null`。*

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改** 链表。

 

**示例 1：**

![image.png](https://s2.loli.net/2025/02/18/nRFy9ZtdaK8fHNv.png)

```
输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。
```

**示例 2：**

![image.png](https://s2.loli.net/2025/02/18/CrTm6S8bGx5ilIN.png)

```
输入：head = [1,2], pos = 0
输出：返回索引为 0 的链表节点
解释：链表中有一个环，其尾部连接到第一个节点。
```

**示例 3：**

![image.png](https://s2.loli.net/2025/02/18/bqDX9FMQWHNaEpf.png)

```
输入：head = [1], pos = -1
输出：返回 null
解释：链表中没有环。
```

 

**提示：**

- 链表中节点的数目范围在范围 `[0, 10^4]` 内
- `-10^5 <= Node.val <= 10^5`
- `pos` 的值为 `-1` 或者链表中的一个有效索引



### 思路

和前一道题一样的思路，使用哈希集合来存储已经出现过的节点。我们只用遍历一次链表，边遍历边往哈希集合中添加元素，每次添加都判断一次哈希集合中是否存在这个元素，如果存在就说明有环，直接返回当前节点即为答案。代码如下

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        unordered_set<ListNode*> st;
        ListNode* p = head;
        while (p) {
            if (st.count(p)) return p;
            else st.insert(p); 
            p = p->next;
        }
        return nullptr;
    }
};
```

和上一题一样的，可以用双指针进行空间上的优化，同样定义快慢指针fast和slow，它们起始都位于链表的头部。随后，slow每次向后移动一个位置，而fast向后移动两个位置。如果链表中存在环，则fast最终会与slow在环中相遇。

如下图所示，设链表中环外部分的长度为**a**。slow进入环后，又走了**b**的距离和fast相遇，此时fast已经走完了环的**n**圈，因此他走过的总距离为：**a + n(b + c) + b = a + (n + 1)b + nc**

![image.png](https://s2.loli.net/2025/02/18/TjbfLIugsiVnF8O.png)

根据题意，**任意时刻，fast走过的距离都为slow的2倍**，所以我们有

**a + (n + 1)b + nc = 2(a + b)**，可以得到：**a = c + (n - 1)(b + c)**

有了这个等量关系，我们发现**从相遇点到入环点的距离加上 n - 1 圈的环长，恰好等于头节点到入环点的距离**。

因此，**当发现 slow 和 fast 相遇时**，我们额外使用一个指针p，起始指向链表头部，随后它和slow每次向后移动一个位置，最终它们会在入环点相遇

代码如下

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if (head == nullptr || head->next == nullptr) return nullptr;
        ListNode *fast = head, *slow = head;
        do {
            if (fast == nullptr || fast->next == nullptr) return nullptr;
            fast = fast->next->next;
            slow = slow->next;
        } while (fast != slow);
        ListNode *p = head;
        while (p != slow) {
            p = p->next;
            slow = slow->next;
        }
        return p;
    }
};
```





## 27.合并两个有序链表

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/merge_ex1.jpg)

```
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
```

**示例 2：**

```
输入：l1 = [], l2 = []
输出：[]
```

**示例 3：**

```
输入：l1 = [], l2 = [0]
输出：[0]
```



**提示：**

- 两个链表的节点数目范围是 `[0, 50]`
- `-100 <= Node.val <= 100`
- `l1` 和 `l2` 均按 **非递减顺序** 排列



### 思路

我们定义一个新指针变量，用来存放合并后的链表。至于如何合并，我们可以定义两个指针p1和p2分别遍历原有的两个链表，逐个的比较两个指针指向的节点的值，将更小的节点接到新链表尾部，并将自身指针往后挪。

一直到某个链表走到头，若此时另一个链表还没有遍历完，那么就直接将剩余链表接到尾部。代码如下

**Code1（迭代写法）**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode *p1 = list1, *p2 = list2;
        ListNode *dummy = new ListNode(), *p = dummy;
        while (p1 && p2) {
            if (p1->val < p2->val) {
                p->next = p1;
                p1 = p1->next;
            } else {
                p->next = p2;
                p2 = p2->next;
            }
            p = p->next;
        }
        if (!p1) p->next = p2;
        else p->next = p1;
        return dummy->next;
    }
};
```

**Code2（递归写法）**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        if (list1 == nullptr) {
            return list2;
        } else if (list2 == nullptr) {
            return list1;
        } else if (list1->val < list2->val) {
            list1->next = mergeTwoLists(list1->next, list2);
            return list1;
        } else {
            list2->next = mergeTwoLists(list1, list2->next);
            return list2;
        }
    }
};
```



## 28.两数相加

给你两个 **非空** 的链表，表示两个非负的整数。它们每位数字都是按照 **逆序** 的方式存储的，并且每个节点只能存储 **一位** 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

 

**示例 1：**

![image.png](https://s2.loli.net/2025/02/18/wi4f3Ysh9zj6ClL.png)

```
输入：l1 = [2,4,3], l2 = [5,6,4]
输出：[7,0,8]
解释：342 + 465 = 807.
```

**示例 2：**

```
输入：l1 = [0], l2 = [0]
输出：[0]
```

**示例 3：**

```
输入：l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
输出：[8,9,9,9,0,0,0,1]
```

 

**提示：**

- 每个链表中的节点数在范围 `[1, 100]` 内
- `0 <= Node.val <= 9`
- 题目数据保证列表表示的数字不含前导零



### 思路

直接模拟即可，代码如下

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode *head = nullptr, *tail = nullptr;
        int carry = 0; // 进位值
        while (l1 || l2) {
            int n1 = l1 ? l1->val : 0;
            int n2 = l2 ? l2->val : 0;
            int sum = n1 + n2 + carry;
            if (!head) {
                head = tail = new ListNode(sum % 10);
            } else {
                tail->next = new ListNode(sum % 10);
                tail = tail->next;
            }
            carry = sum / 10;
            if (l1) l1 = l1->next;
            if (l2) l2 = l2->next;
        }
        // 处理最末尾的数据，若大于10，则创建一个新节点
        if (carry > 0) tail->next = new ListNode(carry);
        return head;
    }
};
```





## 29.删除链表的倒数第N个结点

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/remove_ex1.jpg)

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

**示例 2：**

```
输入：head = [1], n = 1
输出：[]
```

**示例 3：**

```
输入：head = [1,2], n = 1
输出：[1]
```

 

**提示：**

- 链表中结点的数目为 `sz`
- `1 <= sz <= 30`
- `0 <= Node.val <= 100`
- `1 <= n <= sz`



### 思路

删除倒数第N个结点就是**删除正数第(len + 1 - N)**个结点，可以先遍历一遍链表获取链表长度，再遍历一次链表找到第(len - N)个结点，将该结点的next置为next->next删除对应节点，返回head即可。

需要注意两个细节：

- 如果链表长度为1，则删除后只剩下空链表，返回nullptr；
- 如果倒数第N个的这个N == len，则直接将head = head->next，返回即可，否则按照上述操作会得到错误结果

代码如下

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        int len = 0;
        ListNode *p = head;
        // 获取链表长度
        while (p != nullptr) {
            len++;
            p = p->next;
        }
        if (len == 1)
            return nullptr;
        if (len == n) {
            head = head->next;
            return head;
        }

        int cnt = 0;
        p = head;
        while (p != nullptr) {
            cnt++;
            if (cnt == len - n) {
                p->next = p->next->next;
                break;
            }
            p = p->next;
        }
        return head;
    }
};
```

上面做法的两个特殊判断可以通过**添加哑结点**来省去，**即添加一个空结点，该结点指向链表的头节点**。优化如下

```c++
class Solution {
public:
    int getLength(ListNode* head) {
        int length = 0;
        while (head) {
            ++length;
            head = head->next;
        }
        return length;
    }

    ListNode* removeNthFromEnd(ListNode* head, int n) {
        // 添加哑节点指向链表头节点
        ListNode* dummy = new ListNode(0, head);
        int length = getLength(head);
        ListNode* cur = dummy;
        for (int i = 1; i < length - n + 1; ++i) {
            cur = cur->next;
        }
        cur->next = cur->next->next;
        ListNode* ans = dummy->next;
        delete dummy;
        return ans;
    }
};
```



以上方法对给定的链表进行了两次遍历，其实我们可以用双指针，只遍历一次链表得到最终结果

### 双指针只遍历一次链表方法

**我们创建两个指针first和second，first比second超前n个节点，当first到达链表尾部时，second恰好处于倒数第n个节点。**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        if (!head->next) return nullptr;
        ListNode *dummy = new ListNode(0, head);
        ListNode *slow = dummy, *fast = head;
        while (n--) {
            fast = fast->next;
        }
        while (fast) {
            slow = slow->next;
            fast = fast->next;
        }
        slow->next = slow->next->next;
        // 注意此处不能直接返回head，因为有可能删除的就是head节点
        return dummy->next;
    }
};
```





## 30.两两交换链表中的节点

给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/swap_ex1.jpg)

```
输入：head = [1,2,3,4]
输出：[2,1,4,3]
```

**示例 2：**

```
输入：head = []
输出：[]
```

**示例 3：**

```
输入：head = [1]
输出：[1]
```

 

**提示：**

- 链表中节点的数目在范围 `[0, 100]` 内
- `0 <= Node.val <= 100`



### 思路

思路很直接，迭代模拟即可。代码如下

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        ListNode *dummy = new ListNode(0, head);
        ListNode *pre = dummy;
        while (pre->next != nullptr && pre->next->next != nullptr) {
            ListNode *p1 = pre->next;
            ListNode *p2 = pre->next->next;
            pre->next = p2;
            p1->next = p2->next;
            p2->next = p1;
            pre = p1;
        }
        return dummy->next;
    }
};
```





## 31.K个一组翻转链表

给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex1.jpg)

```
输入：head = [1,2,3,4,5], k = 2
输出：[2,1,4,3,5]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex2.jpg)

```
输入：head = [1,2,3,4,5], k = 3
输出：[3,2,1,4,5]
```

 

**提示：**

- 链表中的节点数目为 `n`
- `1 <= k <= n <= 5000`
- `0 <= Node.val <= 1000`



### 思路

本题翻转链表要考虑怎么将反转后的子序列接到其前驱节点和后继节点之间，可结合以下例子理解代码中的反转操作

![1.jpg](https://s2.loli.net/2024/08/05/RoBe9Dznwv2NKb5.jpg) 

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    // 反转一个子链表，返回新的头尾
    pair<ListNode*, ListNode*> myReverse(ListNode *head, ListNode *tail) {
        ListNode *prev = tail->next;
        ListNode *p = head;
        // 翻转子链表
        while (prev != tail) {
            ListNode *nex = p->next;
            // 子序列第一个节点的next要指向子序列的后继节点
            // 所以prev初始化为tail->next
            p->next = prev;
            prev = p;
            p = nex;
        }
        // 翻转后的子序列tail为头节点，head为尾节点
        return {tail, head};
    }
    ListNode* reverseKGroup(ListNode* head, int k) {
        ListNode *dummy = new ListNode(0, head);
        // 子序列前驱节点
        ListNode *pre = dummy;

        while (head) {
            ListNode *tail = pre;

            // 计算链表剩余长度是否大于等于k
            for (int i = 0; i < k; ++i) {
                tail = tail->next;
                // 若不够k，直接返回答案
                if (!tail) return dummy->next;
            }

            // 子序列的后继节点
            ListNode *nex = tail->next;

            // 接收反转后的首尾节点
            tie(head, tail) = myReverse(head, tail);

            // 将子链表接回去
            pre->next = head;
            tail->next = nex;
            pre = tail;
            head = tail->next;
        }
        return dummy->next;
    }
};
```

另解：

```c++
#include<bits/stdc++.h>
using namespace std;

struct ListNode {
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

ListNode* solve(ListNode *head, int k) {
    // 翻转子链表
    auto reverse = [&](ListNode *head) -> ListNode* {
        ListNode *pre = nullptr;
        ListNode *cur = head;
        while (cur != nullptr) {
            ListNode *next = cur->next;
            cur->next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    };

    ListNode *dummy = new ListNode(0);
    dummy->next = head;

    ListNode *pre = dummy;
    ListNode *end = dummy;

    while (end->next != nullptr) {
        // 往后找 k 个节点
        for (int i = 0; i < k && end != nullptr; ++i) {
            end = end->next;
        }
        // 如果不足 k 个，不做任何处理
        if (end == nullptr) break;
        ListNode *start = pre->next; // 待翻转子链表的头部
        ListNode *next = end->next; // 待翻转子链表的后续链表
        end->next = nullptr; // 将子链表拿出来
        pre->next = reverse(start);
        start->next = next;
        pre = start;
        end = pre;
    }
    return dummy->next;
}

int main() {
    // test cases

}
```



## 32.随机链表的复制

给你一个长度为 `n` 的链表，每个节点包含一个额外增加的随机指针 `random` ，该指针可以指向链表中的任何节点或空节点。

构造这个链表的 **[深拷贝](https://baike.baidu.com/item/深拷贝/22785317?fr=aladdin)**。 深拷贝应该正好由 `n` 个 **全新** 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 `next` 指针和 `random` 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。**复制链表中的指针都不应指向原链表中的节点** 。

例如，如果原链表中有 `X` 和 `Y` 两个节点，其中 `X.random --> Y` 。那么在复制链表中对应的两个节点 `x` 和 `y` ，同样有 `x.random --> y` 。

返回复制链表的头节点。

用一个由 `n` 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 `[val, random_index]` 表示：

- `val`：一个表示 `Node.val` 的整数。
- `random_index`：随机指针指向的节点索引（范围从 `0` 到 `n-1`）；如果不指向任何节点，则为 `null` 。

你的代码 **只** 接受原链表的头节点 `head` 作为传入参数。

 

**示例 1：**

![image.png](https://s2.loli.net/2025/02/18/BJbCkxYyTqAuRFf.png)

```
输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
```

**示例 2：**

![image.png](https://s2.loli.net/2025/02/18/SImNfsXkHjYcMKJ.png)

```
输入：head = [[1,1],[2,1]]
输出：[[1,1],[2,1]]
```

**示例 3：**

![image.png](https://s2.loli.net/2025/02/18/QdE5HliNCUxGg1b.png)

```
输入：head = [[3,null],[3,0],[3,null]]
输出：[[3,null],[3,0],[3,null]]
```

 

**提示：**

- `0 <= n <= 1000`
- `-10^4 <= Node.val <= 10^4`
- `Node.random` 为 `null` 或指向链表中的节点。



### 思路

题意如下：

给你一个链表，链表中的每个节点除了**自身 val 值**和**指向下一个节点的 next 指针**，还有一个指向链表中某个节点的**随机指针 random**。题目要我们做的操作是将这个链表完整的复制下来，注意，不是将原来的链表原原本本地复制一遍，是新创建一个链表，**将这个链表的结构仿照原有的列表创建出来。简单来说就是新旧链表对应的每个节点的地址不同**。

那么我们应该如何操作才能完成随机链表的复制呢？我们可以借助将链表转换成线性表的思路，实现对链表的复制。

先将原来的链表转换成线性表，然后我们可以通过哈希表建立映射关系：**节点在线性表中的下标 -> 节点地址**。这样映射之后，我们可以通过线性表的下标访问到节点的 random 所指向的节点。

然后我们再建立另一个映射关系：**当前节点下标 -> random节点下标**。通过这个映射关系，我们可以将旧链表中的 random 关系复制到新链表中。

还要注意有一个细节需要处理：**旧链表中节点的 random 可能指向 nullptr，所以需要特判一下，避免和下标为 0 的节点重复导致过不了样例**。代码如下

```c++
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* next;
    Node* random;
    
    Node(int _val) {
        val = _val;
        next = NULL;
        random = NULL;
    }
};
*/

class Solution {
public:
    Node* copyRandomList(Node* head) {
        int len = 0; // 链表长度
        vector<Node*> List1; // 转换成线性表
        // 将当前节点的地址和下标对应起来
        unordered_map<Node*, int> map1;

        Node *p = head;
        // 创建线性表
        while (p != nullptr) {
            map1.insert({p, len});
            len++;
            List1.push_back(p);
            p = p->next;
        }

        Node *dummy = new Node(0);
        p = dummy;
        Node *q; // 用于创建新节点

        // 第一个是当前下标，第二个是random的下标
        unordered_map<int, int> map2;
        vector<Node*> List2; // 存放新建的链表
        // 创建新链表
        for (int i = 0; i < len; ++i) {
            // 特判random指向空
            if (List1[i]->random == nullptr) {
                map2.insert({i, -1});    
            }
            map2.insert({i, map1[List1[i]->random]});
            // 创建新节点
            q = new Node(List1[i]->val);
            List2.push_back(q);
            // 连接到新链表尾部
            p->next = q;
            p = p->next;
        }

        // 更新新链表中的random
        for (int i = 0; i < len; ++i) {
            if (map2[i] == -1) List2[i]->random = nullptr;
            else List2[i]->random = List2[map2[i]];
        }
        return dummy->next;
    }
};
```

上面的方法使用到了哈希表来记录 random 的相对位置。那么我们能不能使用 `O(1)` 的空间来完成这种复制操作呢？我们可以把新链表和旧链表**混在一起**

例如链表 `1->2->3` ，依次复制每个节点，把新节点直接插到原节点的后面，形成一个交错链表：`1->1'->2->2'->3->3'` 。

如此一来，**原链表节点的下一个节点，就是其对应的新链表节点了**。

然后遍历交错链表，加入节点 1 的 random 指向节点 3，那么就把节点 1' 的 random 指向节点 3 的下一个节点 3'，这样就完成了对 random 指针的复制。

最后从交错链表中分离出 `1'->2'->3'` ，即为深拷贝后的链表。做法类似于[328. 奇偶链表 - 力扣（LeetCode）](https://leetcode.cn/problems/odd-even-linked-list/)

```c++
/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* next;
    Node* random;
    
    Node(int _val) {
        val = _val;
        next = NULL;
        random = NULL;
    }
};
*/

class Solution {
public:
    Node* copyRandomList(Node* head) {
        if (head == nullptr) {
            return nullptr;
        }

        // 复制每个节点，把新节点直接添加到被复制节点的后面
        for (Node *cur = head; cur; cur = cur->next->next) {
            cur->next = new Node(cur->val, cur->next, nullptr);
        }

        for (Node *cur = head; cur; cur = cur->next->next) {
            if (cur->random) {
                // 要复制的 random 是 cur->random 的下一个节点
                cur->next->random = cur->random->next;
            }
        }

        // 分离交错链表
        Node *new_head = head->next;
        Node *cur = head;
        for (; cur->next->next; cur = cur->next) {
            Node *copy = cur->next;
            cur->next = copy->next; // 恢复原节点的 next
            copy->next = cur->next->next; // 设置新节点的 next
        }
        cur->next = nullptr;
        return new_head;
    }
};
```





## 33.排序链表

给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/09/14/sort_list_1.jpg)

```
输入：head = [4,2,1,3]
输出：[1,2,3,4]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/09/14/sort_list_2.jpg)

```
输入：head = [-1,5,3,4,0]
输出：[-1,0,3,4,5]
```

**示例 3：**

```
输入：head = []
输出：[]
```

 

**提示：**

- 链表中节点的数目在范围 `[0, 5 * 10^4]` 内
- `-10^5 <= Node.val <= 10^5`



### 思路

可以用额外 O(n) 的空间，把链表转换成线性表后再 sort，然后再遍历链表依次赋值。代码如下

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        int len = 0;
        ListNode *p = head;
        while (p != nullptr) {
            len++;
            p = p->next;
        }
        p = head;
        vector<int> List(len);
        for (int i = 0; i < len; ++i) {
            List[i] = p->val;
            p = p->next;
        }
        sort(List.begin(), List.end());
        p = head;
        for (auto i : List) {
            p->val = i;
            p = p->next;
        }
        return head;
    }
};
```



能否不用额外的存储空间实现原链表的排序呢？我们可以用**自底向上的归并排序**实现。

归并排序基于分治算法。最容易想到的实现方式是自顶向下的递归实现，考虑到递归调用的栈空间，自顶向下归并排序的空间复杂度是 O(logn) 。如果要达到 O(1) 的空间复杂度，需要用**自底向上**的实现方式。

自底向上的意思是：

- 首先，归并长度为 1 的子链表。例如 `[4,2,1,3]` ，把第一个节点和第二个节点归并，第三个节点和第四个节点归并，得到 `[2,4,1,3]`
- 然后，归并长度为 2 的子链表，例如 `[2,4,1,3]` ，把前两个节点和后两个节点归并，得到 `[1,2,3,4]` 
- 然后，归并长度为 4 的子链表
- 依此类推，直到归并的长度大于等于链表长度为止

具体算法：

1. 遍历链表，获取链表长度 `length`
2. 初始化步长 `step = 1`
3. 循环直到 `step >= length`
4. 每轮循环，从链表头节点开始
5. 分割出两段长为 `step` 的链表，合并，把合并后的链表插到新链表的末尾。重复该步骤，直到链表遍历完比。
6. 把 `step` 扩大一倍，回到第 4 步



代码如下：

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
    // 计算链表长度
    int getListLength(ListNode* head) {
        int length = 0;
        while (head) {
            ++length;
            head = head->next;
        }
        return length;
    }

    // 分割链表
    // 如果链表长度 <= size，直接返回空
    // 如果链表长度 > size，把链表的前 size 个节点分割出来（断开连接），并返回剩余链表的头节点
    ListNode* splitList(ListNode* head, int size) {
        ListNode *cur = head;
        for (int i = 0; i < size - 1 && cur; ++i) {
            cur = cur->next;
        }

        // 链表长度 <= size
        if (cur == nullptr || cur->next == nullptr) {
            return nullptr;
        }

        ListNode *next_head = cur->next;
        cur->next = nullptr;
        return next_head;
    }

    // 合并两个有序链表
    pair<ListNode*, ListNode*> mergeTwoLists(ListNode* list1, ListNode* list2) {
        ListNode dummy;
        ListNode *cur = &dummy; // cur 指向新链表的末尾
        while (list1 && list2) {
            if (list1->val < list2->val) {
                cur->next = list1;
                list1 = list1->next;
            } else {
                cur->next = list2;
                list2 = list2->next;
            }
            cur = cur->next;
        }
        cur->next = list1 ? list1 : list2; // 拼接剩余链表
        while (cur->next) {
            cur = cur->next;
        }
        // 循环结束，cur 是合并后的链表的尾节点
        return {dummy.next, cur};
    }

public:
    ListNode* sortList(ListNode* head) {
        int length = getListLength(head);
        ListNode dummy(0, head);
        // step 为步长，即参与合并的链表长度
        for (int step = 1; step < length; step *= 2) {
            ListNode *new_list_tail = &dummy; // 新链表的末尾
            ListNode *cur = dummy.next;
            while (cur) {
                // 从 cur 开始，分割出两段长为 step 的链表，头节点分别为 head1 和 head2
                ListNode *head1 = cur;
                ListNode *head2 = splitList(head1, step);
                cur = splitList(head2, step); // 按照步长 step 不断分割链表
                // 合并排序两段长为 step 的链表
                auto [head, tail] = mergeTwoLists(head1, head2);
                // 合并后的头节点 head，插到 new_list_tail 的后面
                new_list_tail->next = head;
                new_list_tail = tail;
            }
        }
        return dummy.next;
    }
};
```





## 34.全排列

给定一个不含重复数字的数组 `nums` ，返回其 *所有可能的全排列* 。你可以 **按任意顺序** 返回答案。

 

**示例 1：**

```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

**示例 2：**

```
输入：nums = [0,1]
输出：[[0,1],[1,0]]
```

**示例 3：**

```
输入：nums = [1]
输出：[[1]]
```

 

**提示：**

- `1 <= nums.length <= 6`
- `-10 <= nums[i] <= 10`
- `nums` 中的所有整数 **互不相同**



### 思路

直接搜

```c++
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> res;
        vector<int> temp;
        bool vis[6];
        auto dfs = [&](auto&& dfs, vector<int>& temp) {
            if (temp.size() == n) {
                res.push_back(temp);
                return;
            }
            for (int i = 0; i < n; ++i) {
                if (!vis[i]) {
                    vis[i] = true;
                    temp.push_back(nums[i]);
                    dfs(dfs, temp);
                    vis[i] = false;
                    temp.pop_back();
                }
            }
        };
        dfs(dfs, temp);
        return res;
    }
};
```

从本题中总结一下dfs相关的回溯算法。先看本题的解空间树

![image.png](https://s2.loli.net/2025/02/05/rjUWBG3Zv2lJ5zP.png)

**说明：**

- **每一个节点表示了求解全排列问题的不同的阶段**，这些阶段通过变量的「不同的值」体现，这些变量的不同的值，称之为「状态」；
- 使用深度优先遍历有「回头」的过程，在「回头」以后， **状态变量需要设置成为和先前一样** ，因此在回到上一层结点的过程中，需要撤销上一次的选择，这个操作称之为「状态重置」；
- 深度优先遍历，借助系统栈空间，保存所需要的状态变量，在编码中只需要注意遍历到相应的节点的时候，状态变量的值是正确的，具体的做法是：往下走一层的时候，**path** 变量在尾部追加，而往回走的时候，需要撤销上一次的选择，也是在尾部操作，因此 **path** 变量是一个栈；
- 深度优先遍历通过「回溯」操作，实现了全局使用一份状态变量的效果。

我们通过代码的形式得到全排列，就是在这样的一个**树型结构**中完成**遍历**，从树的根节点到叶子节点形成的路径就是其中一个全排列。



在此之后，我们需要设计dfs的**状态变量**

- 首先，这棵树除了根节点和叶子节点以外，每一个节点**做的事情是一样的**，即：在已经选择了一些数的前提下，在剩下的还没有选择的数中，依次选择一个数，这显然是一个**递归结构**；
- 递归的终止条件：**一个排列中的数字已经选够了**，**因此我们需要一个变量来表示当前递归的层数**，我们把这个变量记为 **depth**；



之后，我们应用遍历排列树的思想。我们将给定 n 个数的数组 nums 划分为左右两部分，左边的表示已经填过的数，右边表示待填的数，我们在回溯的时候只要动态维护这个数组即可。

![image.png](https://s2.loli.net/2025/02/05/ynVuwp6RGFjfLsx.png)

### 补充：全排列Ⅱ

给定一个可包含重复数字的序列 `nums` ，***按任意顺序*** 返回所有不重复的全排列。

 

**示例 1：**

```
输入：nums = [1,1,2]
输出：
[[1,1,2],
 [1,2,1],
 [2,1,1]]
```

**示例 2：**

```
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

 

**提示：**

- `1 <= nums.length <= 8`
- `-10 <= nums[i] <= 10`



#### 思路

与上一题的区别在于可能存在重复元素，以 `nums = {1,1,2}` 为例，在递归的过程中可能会发生如下情况：

1. 第一个位置填 `nums[0]` ，第二个位置填 `nums[1]` 
2. 第一个位置填 `nums[1]` ，第二个位置填 `nums[0]` 

这两种填法都会得到 `[1,1,2]` ，导致重复

要保证第二种情况不会发生：在填 `nums[1] = 1` 时发现 `nums[0]` **还没填过**，那么就直接 continue 。

推广到更多数时，比如：`nums = {1,1,1,2}` ，它有四个排列：

- `[1,1,1,2]`
- `[1,1,2,1]`
- `[1,2,1,1]`
- `[2,1,1,1]`

第一个位置要么填 1 ，要么填 2

**其中第一个位置填 1 的三个排列，会在第一个位置填 `nums[0]` 时枚举到，如果第一个位置填 `nums[1]` 或者 `nums[2]` ，那么必然会产生重复的排列**

所以 `nums[1]` 和 `nums[2]` 绝对不能填到第一个位置上

这意味着，如果有多个 `nums[i]` 相同，我们只需要枚举其中一个 `nums[i]` 填第一个位置的情况，其余所有等于 `nums[i]` 的数都不能填到第一个位置

如果 `nums[0]` 填在了第一个位置，那么问题变成：剩余的 `[1,1,2]` 怎么填？

**这是一个和原问题相似，规模更小的子问题**，处理方式同上：`nums[1]` 可以填在排列的第二个位置，而 `nums[2]` 不能填在排列的第二个位置，否则会导致重复的排列。

**怎么判断？**如果我们还没有填入 `nums[1]` ，那么和 `nums[1]` 相等的 `nums[2]` 是不能填入的。

**那么怎么判断 `nums[i]` 能不能填？**

为了方便判断，先将 `nums` 排序，分类讨论：

- 如果 `nums[i] != nums[i-1]` ，那么 `nums[i]` 就是所有等于 `nums[i]` 的数中的第一个数，这种情况可以随便填
- 如果 `nums[i] == nums[i-1]` ，继续讨论：
  - 如果 `nums[i-1]` 没有填入排列，为了避免重复排列，绝对不能填 `nums[i]` ，直接 continue 
  - 如果 `nums[i-1]` 已经填入排列，那么 `nums[i]` 是剩余元素（子问题）中的第一个等于 `nums[i]` 的数，可以随便填

```c++
#include<bits/stdc++.h>
using namespace std;

vector<vector<int>> solve(vector<int> nums) {
    ranges::sort(nums);
    int n = nums.size();
    vector<vector<int>> res;
    vector<int> path(n); // 所有排列的长度都是 n
    vector<int> on_path(n); // on_path[j] 表示 nums[j] 是否已经填入排列
    auto dfs = [&] (auto&& dfs, int i) {
        if (i == n) {
            res.push_back(path);
            return;
        }
        // 枚举 nums[j] 填入 path[i]
        for (int j = 0; j < n; ++j) {
            // 如果 nums[j] 已经填入了排列 或者 nums[j] == nums[j-1] 并且 nums[j-1] 没有填入排列
            // 就直接跳过
            if (on_path[j] || j > 0 && nums[j] == nums[j-1] && !on_path[j-1]) {
                continue;
            }
            path[i] = nums[j];
            on_path[j] = true;
            dfs(dfs, i + 1);
            on_path[j] = false;
            // path[i] 不需要恢复，可以直接覆盖
        }
    };
    dfs(dfs, 0);
    return res;
}

void print(vector<vector<int>> nums) {
    for (auto x : nums) {
        for (auto y : x) {
            cout << y << " ";
        }
        cout << endl;
    }
    cout << endl;
}

int main() {
    vector<int> nums1 = {1,1,2};
    auto res1 = solve(nums1);
    vector<int> nums2 = {1,2,3};
    auto res2 = solve(nums2);
    print(res1);
    print(res2);
    return 0;
}
```



## 35.子集

给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。

解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。

 

**示例 1：**

```
输入：nums = [1,2,3]
输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

**示例 2：**

```
输入：nums = [0]
输出：[[],[0]]
```

 

**提示：**

- `1 <= nums.length <= 10`
- `-10 <= nums[i] <= 10`
- `nums` 中的所有元素 **互不相同**



### 思路

上一题是排列树，这一题是子集树。从根节点开始往下遍历子集树，对于每个节点都有两种选择，**加入到子集中和不加入到子集中**，我们另开一个数组 t 来记录这个子集。如果遍历到了叶子节点就返回结果。代码如下

```c++
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<vector<int>> res;
        vector<int> temp;
        int n = nums.size(), cur = 0;
        auto dfs = [&](auto&& dfs, vector<int>& temp, int cur) {
            if (cur == nums.size()) {
                res.push_back(temp);
                return;
            }

            // 选当前的数
            temp.push_back(nums[cur]);
            dfs(dfs, temp, cur + 1);
            temp.pop_back();

            // 不选当前的数
            dfs(dfs, temp, cur + 1);
        };
        dfs(dfs, temp, cur);
        return res;
    }
};
```

### 补充：子集Ⅱ

给你一个整数数组 `nums` ，其中可能包含重复元素，请你返回该数组所有可能的子集（幂集）。

解集 **不能** 包含重复的子集。返回的解集中，子集可以按 **任意顺序** 排列。

 

**示例 1：**

```
输入：nums = [1,2,2]
输出：[[],[1],[1,2],[1,2,2],[2],[2,2]]
```

**示例 2：**

```
输入：nums = [0]
输出：[[],[0]]
```

 

**提示：**

- `1 <= nums.length <= 10`
- `-10 <= nums[i] <= 10`



#### 思路

本题和上一题类似，需要注意的是，由于集合可能包含重复元素，在不选 `nums[i]` 时，要跳过后续所有等于 `nums[i]` 的数（这就需要首先对数组进行排序，方便直接跳过所有重复元素）。如果不跳过这些数，设 `x = nums[i]` ，`x' = nums[i+1]` ，那么 **选 x 不选 x'** 和 **不选 x 选 x'** 这两种情况就都会加到答案中，出现重复。

```c++
#include<bits/stdc++.h>
using namespace std;

vector<vector<int>> solve(vector<int>& nums) {
    ranges::sort(nums);
    int n = nums.size();
    vector<vector<int>> res;
    vector<int> temp;
    unordered_set<int> st;
    auto dfs = [&](auto&& dfs, int i) {
        if (i == n) {
            res.push_back(temp);
            return;
        }
        
        int x = nums[i];
        temp.push_back(x);
        dfs(dfs, i + 1);
        temp.pop_back();

        ++i;
        while (i < n && nums[i] == x) {
            ++i;
        }
        dfs(dfs, i);
    };
    dfs(dfs, 0);
    return res;
}

void print(vector<vector<int>>& res) {
    for (auto& x : res) {
        for (auto& y : x) {
            cout << y << " ";
        }
        cout << endl;
    }
}

int main() {
    vector<int> nums1 = {1,2,2};
    vector<int> nums2 = {1,2,3};
    vector<int> nums3 = {0};
    vector<int> nums4 = {1,2,3,4,4,4,5};
    auto res1 = solve(nums1);
    auto res2 = solve(nums2);
    auto res3 = solve(nums3);
    auto res4 = solve(nums4);
    print(res1);
    print(res2);
    print(res3);
    print(res4);
    return 0;
}
```





## 36.电话号码的字母组合

给定一个仅包含数字 `2-9` 的字符串，返回所有它能表示的字母组合。答案可以按 **任意顺序** 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![image.png](https://s2.loli.net/2025/02/05/PjFu72GJAI68lsv.png) 

 

**示例 1：**

```
输入：digits = "23"
输出：["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

**示例 2：**

```
输入：digits = ""
输出：[]
```

**示例 3：**

```
输入：digits = "2"
输出：["a","b","c"]
```

 

**提示：**

- `0 <= digits.length <= 4`
- `digits[i]` 是范围 `['2', '9']` 的一个数字。



### 思路

一道典型的dfs，主要是要确定dfs函数中的参数意义。代码如下

```c++
class Solution {
public:
    void dfs(string& temp, vector<string>& ans, int index, const string& digits, const unordered_map<char, string>& map) {
        if (index == digits.length()) {
            ans.push_back(temp);
            return;
        }

        char digit = digits[index]; // 当前数字
        const string& letters = map.at(digit); // 当前数字对应的string
        for (const char& letter : letters) { // 遍历这个string
            temp.push_back(letter);
            dfs(temp, ans, index + 1, digits, map);
            temp.pop_back(); // 回溯
        }
    }
    vector<string> letterCombinations(string digits) {
        if (digits.size() == 0) return {};
        vector<string> ans;
        string temp = "";
        unordered_map<char, string> map;
        map.insert({'2', "abc"});
        map.insert({'3', "def"});
        map.insert({'4', "ghi"});
        map.insert({'5', "jkl"});
        map.insert({'6', "mno"});
        map.insert({'7', "pqrs"});
        map.insert({'8', "tuv"});
        map.insert({'9', "wxyz"});
        dfs(temp, ans, 0, digits, map);
        return ans;
    }
};
```





## 37.组合总和

给你一个 **无重复元素** 的整数数组 `candidates` 和一个目标整数 `target` ，找出 `candidates` 中可以使数字和为目标数 `target` 的 所有 **不同组合** ，并以列表形式返回。你可以按 **任意顺序** 返回这些组合。

`candidates` 中的 **同一个** 数字可以 **无限制重复被选取** 。如果至少一个数字的被选数量不同，则两种组合是不同的。 

对于给定的输入，保证和为 `target` 的不同组合数少于 `150` 个。

 

**示例 1：**

```
输入：candidates = [2,3,6,7], target = 7
输出：[[2,2,3],[7]]
解释：
2 和 3 可以形成一组候选，2 + 2 + 3 = 7 。注意 2 可以使用多次。
7 也是一个候选， 7 = 7 。
仅有这两种组合。
```

**示例 2：**

```
输入: candidates = [2,3,5], target = 8
输出: [[2,2,2,2],[2,3,3],[3,5]]
```

**示例 3：**

```
输入: candidates = [2], target = 1
输出: []
```

 

**提示：**

- `1 <= candidates.length <= 30`
- `2 <= candidates[i] <= 40`
- `candidates` 的所有元素 **互不相同**
- `1 <= target <= 40`



### 思路

本题也是深搜就能过，熟练深搜的框架就好了，注意本题搜索解空间树时可以剪枝。代码如下

```c++
class Solution {
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        int n = candidates.size();
        vector<vector<int>> res;
        auto dfs = [&](auto&& dfs, int target, int sum, int start, vector<int> temp) {
            if (sum == target) {
                res.push_back(temp);
                return;
            }
            for (int i = start; i < n; ++i) {
                if (sum + candidates[i] > target) continue;
                temp.push_back(candidates[i]);
                dfs(dfs, target, sum + candidates[i], i, temp);
                temp.pop_back();
            }
        };

        dfs(dfs, target, 0, 0, {});
        return res;
    }
};
```





## 38.括号生成

数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 **有效的** 括号组合。

 

**示例 1：**

```
输入：n = 3
输出：["((()))","(()())","(())()","()(())","()()()"]
```

**示例 2：**

```
输入：n = 1
输出：["()"]
```

 

**提示：**

- `1 <= n <= 8`



### 思路

该题的数据量比较小，直接 dfs 就能过。需要注意一点：**最终的结果一定是以左括号开头，右括号结尾，所以必须先对左括号 dfs，再对右括号 dfs。同时右括号 dfs 时要注意判断右括号数量是否小于左括号，小于才能 dfs，否则不行。**代码如下

```c++
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> res;
        auto dfs = [&](auto&& dfs, int front, int back, string temp) {
            if (front == n && back == n) {
                res.push_back(temp);
                return;
            }
            if (front < n) {
                temp.push_back('(');
                dfs(dfs, front + 1, back, temp);
                temp.pop_back();
            } 
            if (back < n && back < front) {
                temp.push_back(')');
                dfs(dfs, front, back + 1, temp);
                temp.pop_back();
            }
        };
        dfs(dfs, 0, 0, "");
        return res;
    }
};
```





## 39.单词搜索

给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/04/word2.jpg)

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
输出：true
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/11/04/word-1.jpg)

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "SEE"
输出：true
```

**示例 3：**

![img](https://assets.leetcode.com/uploads/2020/10/15/word3.jpg)

```
输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCB"
输出：false
```

 

**提示：**

- `m == board.length`
- `n = board[i].length`
- `1 <= m, n <= 6`
- `1 <= word.length <= 15`
- `board` 和 `word` 仅由大小写英文字母组成



### 思路

遍历整个二维网格，找到第一个和 word 首字母相同的元素就开始 dfs，注意判断出界、不匹配的情况。代码如下

```c++
class Solution {
private:
    vector<pair<int, int>> DIRS = { {-1, 0}, {0, -1}, {0, 1}, {1, 0} };
public:
    bool exist(vector<vector<char>>& board, string word) {
        int m = board.size(), n = board[0].size();
        int len = word.length();
        // 特判单个字符
        if (m == 1 && n == 1 && len == 1) return word[0] == board[0][0];
        bool ok = false;
        auto dfs = [&](auto&& dfs, string temp, int cur_len, int x, int y) {
            if (cur_len == len) {
                ok = true;
                return;
            }
            if (board[x][y] != '0') {
                char c = board[x][y];
                if (board[x][y] == word[cur_len]) {
                    temp.push_back(board[x][y]);
                    board[x][y] = '0';
                } else {
                    return;
                }
                for (int i = 0; i < 4; ++i) {
                    int dx = x + DIRS[i].first;
                    int dy = y + DIRS[i].second;
                    if (dx >= 0 && dx < m && dy >= 0 && dy < n) {
                        dfs(dfs, temp, cur_len + 1, dx, dy);
                    }
                }
                board[x][y] = c;
            }
        };
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (board[i][j] != word[0]) continue;
                dfs(dfs, "", 0, i, j);
            }
        }
        return ok;
    }
};
```





## 40.分割回文串

给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是 

**回文串**。返回 `s` 所有可能的分割方案。

 

**示例 1：**

```
输入：s = "aab"
输出：[["a","a","b"],["aa","b"]]
```

**示例 2：**

```
输入：s = "a"
输出：[["a"]]
```

 

**提示：**

- `1 <= s.length <= 16`
- `s` 仅由小写英文字母组成



### 思路

假设当前我们搜索到字符串的第 i 个字符，且 s[0...i-1] 的所有字符已经分好了，那么就只用枚举下一个回文串的右边界 j，使得 s[i...j] 是一个回文串。

所以我们可以从 i 开始，从小到大依次枚举 j，对于当前枚举的 j 值，用双指针的方法判断 s[i...j] 是否为回文串。如果是回文串就加到答案数组中，并以 j + 1 作为新的 i 进行下一层搜索。

当我们判断回文串时，用双指针会导致重复计算。比如下面这个例子：

当 s = aaba 时，对于前2个字符aa，我们有两种分割方法：**[aa]和[a, a]**。当我们每一次搜索到字符串的第 i = 2 个字符 b 时，都需要对每个 s[i...j] 使用双指针判断其是否为回文串，就产生了重复计算。

所以对于回文串的判断，可以用dp预处理出来。

代码如下

```c++
class Solution {
private:
    // dp数组，用于判断是否为回文串
    // 若f[i][j] == true，则s[i...j]为回文串
    vector<vector<bool>> f;
    vector<vector<string>> ret; // 答案数组
    vector<string> ans; // 存放回文子串
    int n; // s串长度

public:
    void dfs(const string& s, int i) {
        if (i == n) {
            ret.push_back(ans);
            return;
        }
        // 从上一个回文串的下一位开始
        for (int j = i; j < n; ++j) {
            if (f[i][j]) {
                // 获取回文子串
                ans.push_back(s.substr(i, j - i + 1));
                dfs(s, j + 1);
                ans.pop_back(); // 回溯
            }
        }
    }

    vector<vector<string>> partition(string s) {
        n = s.size();
        // 初始化f数组
        f.assign(n, vector<bool>(n, true));

        // 动态规划，先判断所有的子串是否为回文串
        for (int i = n - 1; i >= 0; --i) { // 找子串从后往前遍历会比较方便
            for (int j = i + 1; j < n; ++j) {
                // 当f[i + 1][j - 1]为回文且s[i] == s[j]，则f[i][j]回文
                f[i][j] = (s[i] == s[j]) && f[i + 1][j - 1];
            }
        }

        dfs(s, 0);
        return ret;
    }
};
```

另解：回溯枚举

```c++
class Solution {
public:
    vector<vector<string>> partition(string s) {
        auto is_reverse = [&](string& s, int left, int right) -> bool {
            while (left < right) {
                if (s[left++] != s[right--]) {
                    return false;
                }
            }
            return true;
        };

        int len = s.length();
        vector<vector<string>> ans;
        vector<string> path;

        auto dfs = [&](auto&& dfs, int i) {
            if (i == len) {
                ans.emplace_back(path);
                return;
            }
            for (int j = i; j < len; ++j) { // 枚举字串结束的位置
                if (is_reverse(s, i, j)) {
                    path.push_back(s.substr(i, j - i + 1));
                    dfs(dfs, j + 1);
                    path.pop_back();
                }
            }
        };

        dfs(dfs, 0);
        return ans;
    }
};
```



## 41.N皇后

按照国际象棋的规则，皇后可以攻击与之处在同一行或同一列或同一斜线上的棋子。

**n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n×n` 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 `n` ，返回所有不同的 **n 皇后问题** 的解决方案。

每一种解法包含一个不同的 **n 皇后问题** 的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/13/queens.jpg)

```
输入：n = 4
输出：[[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
解释：如上图所示，4 皇后问题存在两个不同的解法。
```

**示例 2：**

```
输入：n = 1
输出：[["Q"]]
```

 

**提示：**

- `1 <= n <= 9`



### 思路

经典的N皇后问题，使用枚举即可。行列枚举比较容易，主要是怎么枚举对角线上的元素。

**对于左上到右下的对角线，对角线上的元素行列坐标之差为定值；对于左下到右上的对角线，对角线上的元素行列坐标之和为定值**。根据这个性质进行枚举搜索即可。代码如下

```c++
class Solution {
public:
    vector<vector<string>> solveNQueens(int n) {
        // 生成答案
        auto generateBoard = [&](vector<int>& queens, int n) -> vector<string> {
            vector<string> board;
            for (int i = 0; i < n; ++i) {
                string row = string(n, '.');
                row[queens[i]] = 'Q';
                board.push_back(row);
            }
            return board;
        };

        vector<vector<string>> ans;
        vector<int> queens = vector<int>(n, -1);
        unordered_set<int> col, diag1, diag2;
        auto dfs = [&](auto&& dfs, int n, int row) {
            if (row == n) {
                auto board = generateBoard(queens, n);
                ans.push_back(board);
                return;
            }
            for (int i = 0; i < n; ++i) {
                if (col.contains(i)) continue;
                int diagnal1 = row - i;
                if (diag1.contains(diagnal1)) continue;
                int diagnal2 = row + i;
                if (diag2.contains(diagnal2)) continue;
                queens[row] = i;
                col.insert(i);
                diag1.insert(diagnal1);
                diag2.insert(diagnal2);
                dfs(dfs, n, row + 1);
                col.erase(i);
                diag1.erase(diagnal1);
                diag2.erase(diagnal2);
            }
        };

        dfs(dfs, n, 0);
        return ans;
    }
};
```





## 42.合并K个升序链表

给你一个链表数组，每个链表都已经按升序排列。

请你将所有链表合并到一个升序链表中，返回合并后的链表。

 

**示例 1：**

```
输入：lists = [[1,4,5],[1,3,4],[2,6]]
输出：[1,1,2,3,4,4,5,6]
解释：链表数组如下：
[
  1->4->5,
  1->3->4,
  2->6
]
将它们合并到一个有序链表中得到。
1->1->2->3->4->4->5->6
```

**示例 2：**

```
输入：lists = []
输出：[]
```

**示例 3：**

```
输入：lists = [[]]
输出：[]
```

 

**提示：**

- `k == lists.length`
- `0 <= k <= 10^4`
- `0 <= lists[i].length <= 500`
- `-10^4 <= lists[i][j] <= 10^4`
- `lists[i]` 按 **升序** 排列
- `lists[i].length` 的总和不超过 `10^4`



### 思路

转换成线性表，用额外 `O(n)` 的空间进行排序，再创建新的链表填入数据。

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        vector<int> nums;
        for (auto& list: lists) {
            ListNode *p = list;
            while (p) {
                nums.push_back(p->val);
                p = p->next;
            }
        }
        ranges::sort(nums);
        ListNode *dummy = new ListNode(0, nullptr), *p = dummy;
        for (int& num: nums) {
            ListNode *new_node = new ListNode(num);
            p->next = new_node;
            p = p->next;
        }
        return dummy->next;
    }
};
```

**分治合并思路**

- 将k个链表配对，并将同一对链表合并；
- 第一轮合并后，k个链表被合并成了k/2个链表，平均长度为：n / (k/2) = 2n/k。然后是k/4个链表、k/8个链表等等
- 重复这一过程，直到最终得到有序链表

![image.png](https://s2.loli.net/2025/02/11/AYh6pdwWZD4lfm2.png)

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    // 合并两个链表
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
        if ((!list1) || (!list2)) return list1 ? list1 : list2;
        ListNode head, *tail = &head, *p1 = list1, *p2 = list2;
        while (p1 && p2) {
            if (p1->val < p2->val) {
                tail->next = p1;
                p1 = p1->next;
            } else {
                tail->next = p2;
                p2 = p2->next;
            }
            tail = tail->next;
        }
        tail->next = (p1 ? p1 : p2);
        return head.next;
    }

    ListNode* merge(vector<ListNode*> &lists, int l, int r) {
        // 当子问题已经被分解到可以解决，返回
        if (l == r) return lists[l];
        if (l > r) return nullptr;
        int mid = (l + r) >> 1; // 右移相当于除2
        // 分治
        return mergeTwoLists(merge(lists, l, mid), merge(lists, mid + 1, r));
    }

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        return merge(lists, 0, lists.size() - 1);
    }
};
```

除了上述两种方法，对于本题还可以使用**最小堆**来实现。

由于原来给出的链表数组中，每个链表都已经按升序排列，因此合并后的链表的头节点一定是数组中某个链表的头节点。

那么合并后链表的下一个节点，可能是其他链表的头节点，也可能是当前链表的第二个节点。我们怎么判断呢？可以通过**最小堆**来实现。

首先创建一个最小堆，往里面存放每个链表的头节点。由于最小堆的特性，所有的头节点均按照从小到大的顺序排好。然后将堆顶元素弹出，加入到合并的链表中，如果弹出节点的 next 不为空，那么就将 next 加入到最小堆。

重复上述操作，直到堆为空，我们就完成了 K 个升序链表的合并。

**Code**

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto cmp = [](const ListNode *a, const ListNode *b) {
            return a->val > b->val; // 最小堆
        };
        priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq;
        for (auto head: lists) {
            if (head) {
                pq.push(head);
            }
        }

        ListNode dummy{};
        auto cur = &dummy;
        while (!pq.empty()) {
            auto node = pq.top();
            pq.pop();
            if (node->next) {
                pq.push(node->next);
            }
            cur->next = node;
            cur = cur->next;
        }
        return dummy.next;
    }
};
```

**需要注意：下面的代码表示最小堆，cmp 函数中 `a->val > b->val` 表示如果 a 的值比 b 大，那么 a 的优先级更低**。

```c++
auto cmp = [](const ListNode *a, const ListNode *b) {
    return a->val > b->val;
};
priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq;
```





## 43.LRU缓存

请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

 

**示例：**

```
输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```

 

**提示：**

- `1 <= capacity <= 3000`
- `0 <= key <= 10000`
- `0 <= value <= 10^5`
- 最多调用 `2 * 10^5` 次 `get` 和 `put`



### 思路

![img](https://pic.leetcode.cn/1696039105-PSyHej-146-3-c.png)

```c++
#include<bits/stdc++.h>
using namespace std;

class Node {
    public:
        int key;
        int value;
        Node *prev;
        Node *next;

        Node(int k = 0, int v = 0) : key(k), value(v) {}
};

class LRUCache {
    private:
        int capacity;
        Node *dummy;
        unordered_map<int, Node*> key_to_node;

        // 删除双向链表中的一个节点
        void remove(Node *x) {
            x->prev->next = x->next;
            x->next->prev = x->prev;
        }

        // 链表头部添加一个节点
        void push_front(Node *x) {
            x->prev = dummy;
            x->next = dummy->next;
            x->next->prev = x;
            x->prev->next = x;
        }

        // 获取 key 对应节点，并放到链表头部
        Node* get_node(int key) {
            auto it = key_to_node.find(key);
            // 没有这本书
            if (it == key_to_node.end()) {
                return nullptr;
            }
            // 有这本书
            Node *node = it->second;
            remove(node); // 把这本书抽出来
            push_front(node); // 放到最上面
            return node;
        }

    public:
        LRUCache(int capacity) : capacity(capacity), dummy(new Node()) {
            dummy->prev = dummy;
            dummy->next = dummy;
        }

        int get(int key) {
            Node *node = get_node(key);
            return node ? node->value : -1;
        }

        void put(int key, int value) {
            Node *node = get_node(key);
            if (node) { // 有这本书
                node->value = value;
                return;
            }
            // 没有这本书
            key_to_node[key] = node = new Node(key, value);
            push_front(node); // 放到最上面
            if (key_to_node.size() > capacity) {
                Node *back_node = dummy->prev;
                // 哈希表中删除
                key_to_node.erase(back_node->key);
                // 双向链表中删除
                remove(back_node);
                delete back_node;
            }
        }
};


int main() {
    LRUCache* obj = new LRUCache(2);
    obj->put(1, 1);
    obj->put(2, 2);
    int value = obj->get(1);
    cout << "key:1, value:" << value << endl;
    obj->put(3, 3);
    value = obj->get(2);
    cout << "key:2, value:" << value << endl;
    obj->put(4, 4);
    value = obj->get(1);
    cout << "key:1, value:" << value << endl;
    value = obj->get(3);
    cout << "key:3, value:" << value << endl;
    value = obj->get(4);
    cout << "key:4, value:" << value << endl;
    return 0;
}
```





## 44.二叉树的中序遍历

给定一个二叉树的根节点 `root` ，返回 *它的 **中序** 遍历* 。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/09/15/inorder_1.jpg)

```
输入：root = [1,null,2,3]
输出：[1,3,2]
```

**示例 2：**

```
输入：root = []
输出：[]
```

**示例 3：**

```
输入：root = [1]
输出：[1]
```

 

**提示：**

- 树中节点数目在范围 `[0, 100]` 内
- `-100 <= Node.val <= 100`



### 中序遍历递归与迭代写法

```c++
#include<bits/stdc++.h>
using namespace std;

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(left) {}
};

// 递归实现中序遍历二叉树
vector<int> solve1(TreeNode *root) {
    vector<int> res;
    auto dfs = [&](auto&& dfs, TreeNode *root) {
        if (!root) return;
        dfs(dfs, root->left);
        res.push_back(root->val);
        dfs(dfs, root->right);
    };
    dfs(dfs, root);
    return res;
}

// 迭代实现中序遍历二叉树
vector<int> solve2(TreeNode *root) {
    vector<int> res;
    stack<TreeNode*> stk;
    TreeNode *cur = root;

    while (cur || stk.size()) {
        // 不断将左节点压栈，直到叶子节点
        while (cur) {
            stk.push(cur);
            cur = cur->left;
        }

        // 访问当前节点
        cur = stk.top();
        stk.pop();
        res.push_back(cur->val);

        // 右子树准备到下一轮循环
        cur = cur->right;
    }

    return res;
}

void print(vector<int> nums) {
    for (int x : nums) cout << x << " ";
    cout << endl;
}

int main() {
    TreeNode *root = new TreeNode(1);
    root->right = new TreeNode(2);
    root->right->left = new TreeNode(3);
    auto res1 = solve1(root);
    auto res2 = solve2(root);
    print(res1);
    print(res2);
    return 0;
}
```



### 补充：前序遍历与后序遍历

前序遍历**递归写法**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        auto dfs = [&](auto&& dfs, TreeNode* root) {
            if (!root) return;
            res.push_back(root->val);
            dfs(dfs, root->left);
            dfs(dfs, root->right);
        };
        dfs(dfs, root);
        return res;
    }
};
```

前序遍历**迭代写法**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        if (!root) return res;

        stack<TreeNode*> stk;
        TreeNode* node = root;
        while (!stk.empty() || node != nullptr) {
            while (node != nullptr) {
                res.push_back(node->val);
                stk.push(node);
                node = node->left;
            }
            node = stk.top();
            stk.pop();
            node = node->right;
        }
        return res;
    }
};
```

后序遍历**递归写法**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> res;
        auto dfs = [&](auto&& dfs, TreeNode* root) {
            if (!root) return;
            dfs(dfs, root->left);
            dfs(dfs, root->right);
            res.push_back(root->val);
        };
        dfs(dfs, root);
        return res;
    }
};
```

后序遍历**迭代写法**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> res;
        if (!root) return res;

        stack<TreeNode*> stk;
        TreeNode* pre = nullptr;
        while (root != nullptr || !stk.empty()) {
            while (root != nullptr) {
                stk.push(root);
                root = root->left;
            }
            root = stk.top();
            stk.pop();
            if (root->right == nullptr || root->right == pre) {
                res.push_back(root->val);
                pre = root;
                root = nullptr;
            } else {
                stk.push(root);
                root = root->right;
            }
        }
        return res;
    }
};
```





## 45.二叉树的最大深度

给定一个二叉树 `root` ，返回其最大深度。

二叉树的 **最大深度** 是指从根节点到最远叶子节点的最长路径上的节点数。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/26/tmp-tree.jpg)

 

```
输入：root = [3,9,20,null,null,15,7]
输出：3
```

**示例 2：**

```
输入：root = [1,null,2]
输出：2
```

 

**提示：**

- 树中节点的数量在 `[0, 104]` 区间内。
- `-100 <= Node.val <= 100`



### 思路

先序遍历二叉树，当遍历到空节点时更新高度答案即可

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int maxDepth(TreeNode* root) {
        int res = 0, temp = 0;
        auto dfs = [&](auto&& dfs, TreeNode* root, int temp) {
            if (!root) {
                res = max(res, temp);
                return;
            }
            ++temp;
            dfs(dfs, root->left, temp);
            dfs(dfs, root->right, temp);
        };
        dfs(dfs, root, temp);
        return res;
    }
};
```

另一种更简洁的代码形式

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) return 0;
        return max(maxDepth(root->left), maxDepth(root->right)) + 1;
    }
};
```

上面两种代码都是`dfs`，这里提供一种`bfs`的做法

我们广度优先搜索的队列里存放的是**当前层的所有节点**。每次拓展下一层的时候，不同于广度优先搜索的每次只从队列里拿出一个节点，**我们需要将队列里的所有节点拿出来进行拓展**，这样能保证**每次拓展完的时候队列里存放的是当前层的所有节点**，即我们是一层一层地进行拓展，最后我们用一个变量`ans`来维护拓展的次数，该二叉树的最大深度即为`ans`。

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) return 0;
        queue<TreeNode*> Q;
        Q.push(root);
        int ans = 0;
        while (!Q.empty()) {
            int sz = Q.size(); // 当前层的节点个数
            while (sz > 0) { // 扩展当前层所有节点的左右节点
                TreeNode* node = Q.front();
                Q.pop();
                if (node->left) Q.push(node->left);
                if (node->right) Q.push(node->right);
                sz -= 1;
            }
            ans ++;
        }
        return ans;
    }
};
```





## 46.翻转二叉树

给你一棵二叉树的根节点 `root` ，翻转这棵二叉树，并返回其根节点。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/03/14/invert1-tree.jpg)

```
输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/03/14/invert2-tree.jpg)

```
输入：root = [2,1,3]
输出：[2,3,1]
```

**示例 3：**

```
输入：root = []
输出：[]
```

 

**提示：**

- 树中节点数目范围在 `[0, 100]` 内
- `-100 <= Node.val <= 100`



### 思路

对于每个节点，我们要做的事只有三个：**1.将以左子节点为根节点的二叉树翻转；2.将以右子节点为根节点的二叉树翻转；3.交换左右子结点。**递归实现即可。

**Code1**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (!root) return nullptr;
        TreeNode* left = invertTree(root->left);
        TreeNode* right = invertTree(root->right);
        root->left = right;
        root->right = left;
        return root;
    }
};
```

**Code2**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        auto dfs = [&](auto&& dfs, TreeNode* root) {
            if (!root) return;
            TreeNode *temp = root->left;
            root->left = root->right;
            root->right = temp;
            if (root->left) dfs(dfs, root->left);
            if (root->right) dfs(dfs, root->right);
        };
        dfs(dfs, root);
        return root;
    }
};
```



## 47.对称二叉树

给你一个二叉树的根节点 `root` ， 检查它是否轴对称。

 

**示例 1：**

![img](https://pic.leetcode.cn/1698026966-JDYPDU-image.png)

```
输入：root = [1,2,2,3,4,4,3]
输出：true
```

**示例 2：**

![img](https://pic.leetcode.cn/1698027008-nPFLbM-image.png)

```
输入：root = [1,2,2,null,3,null,3]
输出：false
```

 

**提示：**

- 树中节点数目在范围 `[1, 1000]` 内
- `-100 <= Node.val <= 100`



### 思路

判断一棵二叉树是否对称，需要用两个指针 p 和 q 同时递归遍历根节点的左右子树。如果 p 和 q 指向的节点值相等，并且 p 的左子树与 q 的右子树相同，p 的右子树和 q 的左子树相同，那么这棵二叉树就是对称的。

注意：当递归到空节点时，需要判断两个指针是否都为空。

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        auto isSame = [&](auto&& isSame, TreeNode *p, TreeNode *q) -> bool {
            if (p == nullptr || q == nullptr) return p == q;
            // 值相等且左右子树分别相等
            return p->val == q->val && isSame(isSame, p->left, q->right) 
                                    && isSame(isSame, p->right, q->left);
        };
        return isSame(isSame, root->left, root->right);
    }
};
```





## 48.二叉树的直径

给你一棵二叉树的根节点，返回该树的 **直径** 。

二叉树的 **直径** 是指树中任意两个节点之间最长路径的 **长度** 。这条路径可能经过也可能不经过根节点 `root` 。

两节点之间路径的 **长度** 由它们之间边数表示。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/03/06/diamtree.jpg)

```
输入：root = [1,2,3,4,5]
输出：3
解释：3 ，取路径 [4,2,1,3] 或 [5,2,1,3] 的长度。
```

**示例 2：**

```
输入：root = [1,2]
输出：1
```

 

**提示：**

- 树中节点数目在范围 `[1, 10^4]` 内
- `-100 <= Node.val <= 100`



### 思路

根据题意，在二叉树中的任意一条路径，都可以看做**以某个节点为根节点的左右子树深度之和**。所以我们以二叉树中的每个节点为根节点，递归搜索其左右子树的最大深度和即可。

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int diameterOfBinaryTree(TreeNode* root) {
        int res = 0;
        auto dfs = [&](auto&& dfs, TreeNode *root) -> int {
            if (!root) return -1; // 当遍历到根节点，返回 -1，再 +1 就得到 0
            int left_max = dfs(dfs, root->left); // 计算左子树链长
            int right_max = dfs(dfs, root->right); // 计算右子树链长
            res = max(res, left_max + right_max + 2); // 左右子树的最大链长+根节点连接左右子树的两条边得到答案
            return max(left_max, right_max) + 1; // 以当前节点为根节点的最大链长
        };
        dfs(dfs, root);
        return res;
    }
};
```

这个递归过程可能有点难理解，我们举一个例子来说明。比如下面的二叉树

![img](https://assets.leetcode.com/uploads/2021/03/06/diamtree.jpg) 



- 从根节点1开始，计算节点2和节点3的深度
- 对于节点2，进一步计算节点4和节点5的深度。节点4和节点5都是叶子节点，其深度为0，因此节点2的深度为`max(0, 0) + 1 = 1`
- 节点2的左右子树深度和为2，更新`ans`为2
- 同理，计算节点3的深度为1
- 最后根节点1的左右子树深度和为`2 + 1 = 3`，最终`ans`为3





## 49.二叉树的层序遍历

给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/02/19/tree1.jpg)

```
输入：root = [3,9,20,null,null,15,7]
输出：[[3],[9,20],[15,7]]
```

**示例 2：**

```
输入：root = [1]
输出：[[1]]
```

**示例 3：**

```
输入：root = []
输出：[]
```

 

**提示：**

- 树中节点数目在范围 `[0, 2000]` 内
- `-1000 <= Node.val <= 1000`



### 思路

简单 BFS

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if (!root) return {};
        vector<vector<int>> res;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            int sz = q.size();
            vector<int> temp;
            for (int i = 0; i < sz; ++i) {
                TreeNode *p = q.front();
                q.pop();
                temp.push_back(p->val);
                if (p->left) q.push(p->left);
                if (p->right) q.push(p->right);
            }
            res.push_back(temp);
        }
        return res;
    }
};
```





## 50.将有序数组转换为二叉搜索树

给你一个整数数组 `nums` ，其中元素已经按 **升序** 排列，请你将其转换为一棵 

平衡

 二叉搜索树。



 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/02/18/btree1.jpg)

```
输入：nums = [-10,-3,0,5,9]
输出：[0,-3,9,-10,null,5]
解释：[0,-10,5,null,-3,null,9] 也将被视为正确答案：
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/02/18/btree.jpg)

```
输入：nums = [1,3]
输出：[3,1]
解释：[1,null,3] 和 [3,1] 都是高度平衡二叉搜索树。
```

 

**提示：**

- `1 <= nums.length <= 104`
- `-104 <= nums[i] <= 104`
- `nums` 按 **严格递增** 顺序排列



### 思路

二叉搜索树的特性是：**树的中序遍历是升序序列**。

题目中给定的数组是按照升序排序的有序数组，因此可以确保数组是二叉搜索树的中序遍历序列，我们要做的是把这棵二叉搜索树创建出来。

给定二叉搜索树的中序遍历，能否唯一的确定二叉搜索树？答案是否定的，如果没有要求二叉搜索树的高度平衡，则任何一个数字都可以作为二叉搜索树的根节点。

![image.png](https://s2.loli.net/2025/02/18/mnF3LQWCxgpcrfM.png) 

那如果增加一个限制条件，要求二叉搜索树的高度平衡，是否可以唯一确定二叉搜索树？仍然不可以。

![image.png](https://s2.loli.net/2025/02/18/tSXlnOJfTdwve3L.png) 

直观上看，我们可以选择中间数字作为二叉搜索树的根节点，这样分给左右子树的数字个数相同或只相差 1，可以使得树保持平衡。**如果数组长度是奇数，则根节点的选择是唯一的**，**如果数组长度是偶数，则可以选择中间位置左边的数字作为根节点或者选择中间位置右边的数字作为根节点**，选择不同的数字作为根节点则创建的平衡二叉搜索树也是不同的。

确定平衡二叉搜索树的根节点之后，其余的数字分别位于平衡二叉搜索树的左子树和右子树中，左子树和右子树分别也是平衡二叉搜索树，因此可以通过**递归**的方式创建平衡二叉搜索树。

**递归的基准情形是平衡二叉搜索树不包含任何数字，此时平衡二叉搜索树为空**。在给定中序遍历序列数组的情况下，每一个子树中的数字在数组中一定是连续的，因此可以通过数组下标范围确定子树包含的数字，下标范围记为`[left, right]`。对于整个中序遍历序列，下标范围从`left = 0`到`right = nums.size() - 1`。当`left > right`时，平衡二叉搜索树为空。

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        auto build = [&](auto&& build, vector<int>& nums, int left, int right) -> TreeNode* {
            if (left > right) return nullptr;
            // 给定数组有序，中间节点即为根节点
            int mid = (left + right) / 2;
            TreeNode *root = new TreeNode(nums[mid]); // 构建根节点
            root->left = build(build, nums, left, mid - 1); // 构建左子树
            root->right = build(build, nums, mid + 1, right); // 构建右子树
            return root;
        };

        if (!nums.size()) return nullptr;
        return build(build, nums, 0, nums.size() - 1);
    }
};
```





## 51.验证二叉搜索树

给你一个二叉树的根节点 `root` ，判断其是否是一个有效的二叉搜索树。

**有效** 二叉搜索树定义如下：

- 节点的左子树只包含小于当前节点的数。

- 节点的右子树只包含 **大于** 当前节点的数。

- 所有左子树和右子树自身必须也是二叉搜索树。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/12/01/tree1.jpg)

```
输入：root = [2,1,3]
输出：true
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/12/01/tree2.jpg)

```
输入：root = [5,1,4,null,null,3,6]
输出：false
解释：根节点的值是 5 ，但是右子节点的值是 4 。
```

 

**提示：**

- 树中节点数目范围在`[1, 10^4]` 内
- `-2^31 <= Node.val <= 2^31 - 1`



### 思路

对于二叉搜索树，需要满足根节点的**左子树上的节点均小于根节点**，**右子树上的节点均大于根节点**

我们可以递归遍历左右子树，将根结点的值分别规定为左右子树的上界与下界，如果遍历到节点的数值不在这个范围内，就说明不是有效的二叉搜索树。

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        auto dfs = [&](auto&& dfs, TreeNode* root, long long lower, long long upper) -> bool {
            if (!root) return true;
            if (root->val <= lower || root->val >= upper) return false;
            return dfs(dfs, root->left, lower, root->val) && dfs(dfs, root->right, root->val, upper);
        };
        return dfs(dfs, root, LONG_MIN, LONG_MAX);
    }
};
```

当然，我们也可以利用二叉搜索树的性质，对给定的树进行中序遍历。因为二叉搜索树中序遍历的结果一定是升序的，所以我们用一个 vector 接收树上的元素，判断这个 vector 是否升序即可。

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        vector<int> nums;
        auto dfs = [&](auto&& dfs, TreeNode* root) {
            if (!root) return;
            dfs(dfs, root->left);
            nums.push_back(root->val);
            dfs(dfs, root->right);
        };
        dfs(dfs, root);
        for (int i = 1; i < nums.size(); ++i) {
            if (nums[i] <= nums[i-1]) {
                return false;
            }
        }
        return true;
    }
};
```



## 52.二叉搜索树中第K小的元素

给定一个二叉搜索树的根节点 `root` ，和一个整数 `k` ，请你设计一个算法查找其中第 `k` 个最小元素（从 1 开始计数）。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/01/28/kthtree1.jpg)

```
输入：root = [3,1,4,null,2], k = 1
输出：1
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/01/28/kthtree2.jpg)

```
输入：root = [5,3,6,2,4,null,null,1], k = 3
输出：3
```

 

 

**提示：**

- 树中的节点数为 `n` 。
- `1 <= k <= n <= 10^4`
- `0 <= Node.val <= 10^4`



### 思路

最简单的思路是遍历一遍二叉树，把值存到数组中然后sort，直接在O(1)时间内找到。

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    void dfs(TreeNode* root, vector<int> &nums) {
        if (!root) return;
        nums.push_back(root->val);
        dfs(root->left, nums);
        dfs(root->right, nums);
    }

    int kthSmallest(TreeNode* root, int k) {
        vector<int> nums;
        dfs(root, nums);
        sort(nums.begin(), nums.end());
        return nums[k - 1];
    }
};
```

根据二叉搜索树的性质，**中序遍历二叉搜索树得到的序列是一个升序序列**，所以我们直接对给定的二叉搜索树进行中序遍历，找到第k小的元素即可。**这里的中序遍历可以通过栈结构实现**

**Code1**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        stack<TreeNode*> stack; // 通过栈实现二叉树中序遍历
        while (root != nullptr || stack.size() > 0) {
            while (root != nullptr) {
                stack.push(root);
                root = root->left;
            }
            root = stack.top();
            stack.pop();
            --k;
            if (k == 0) break;
            root = root->right;
        }
        return root->val;
    }
};
```

我们也可以直接 dfs 实现中序遍历

**Code2**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        vector<int> nums;
        auto dfs = [&](auto&& dfs, TreeNode* root) {
            if (!root) return;
            dfs(dfs, root->left);
            nums.push_back(root->val);
            dfs(dfs, root->right);
        };
        dfs(dfs, root);
        int n = nums.size();
        return nums[k-1];
    }
};
```



### 进一步思考

**1.如果需要频繁地查找第k小的值，可以如何优化算法？**

在上述方法中，我们之所以需要中序遍历前k个元素，是因为不知道子树的节点数量，不得不通过遍历子树的方式来获知。

因此，**我们可以记录下以每个节点为根节点的子树的节点数**，并在查找第k小的元素时使用如下方法搜索：

- 令`node`等于根节点，开始搜索
- 对当前节点`node`进行如下操作：
  - 如果`node`的左子树节点数`left`小于`k - 1`，则第`k`小的元素一定在`node`的右子树中，令`node`等于其右子节点，`k`等于`k - left - 1`，并继续搜索；
  - 如果`node`的左子树的节点数`left`等于`k - 1`，则第`k`小的元素就是`node`；
  - 如果`node`的左子树的节点数`left`大于`k - 1`，则第`k`小的元素一定在`node`的左子树中，令`node`等于其左子节点，并继续搜索

在实现中，**我们既可以将每个节点为根节点的子树的节点数存储在节点中，也可以将其记录在哈希表中**。

```c++
class MyBst {
public:
    MyBst(TreeNode *root) {
        this->root = root;
        countNodeNum(root); // 子树节点数目
    }

    // 返回二叉搜索树中第k小的元素
    int kthSmallest(int k) {
        TreeNode *node = root;
        while (node != nullptr) {
            int left = getNodeNum(node->left);
            if (left < k - 1) {
                node = node->right;
                k -= left + 1;
            } else if (left == k - 1) {
                break;
            } else {
                node = node->left;
            }
        }
        return node->val;
    }

private:
    TreeNode *root;
    unordered_map<TreeNode *, int> nodeNum;

    // 统计以node为根结点的子树的结点数
    int countNodeNum(TreeNode * node) {
        if (node == nullptr) {
            return 0;
        }
        nodeNum[node] = 1 + countNodeNum(node->left) + countNodeNum(node->right);
        return nodeNum[node];
    }

    // 获取以node为根结点的子树的结点数
    int getNodeNum(TreeNode * node) {
        if (node != nullptr && nodeNum.count(node)) {
            return nodeNum[node];
        }else{
            return 0;
        }
    }
};

class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        MyBst bst(root);
        return bst.kthSmallest(k);
    }
};
```



**2.如果二叉搜索树经常被修改（插入/删除操作）并且需要频繁地查找第k小的值，可以如何优化算法？**

**解决方案：手写一个AVL（平衡二叉搜索树）。**

- AVL中每个节点的左右子树高度最多相差1
- AVL的子树也是AVL
- 一棵存有`n`个节点的AVL的高度是`O(logn)`

```c++
// 平衡二叉搜索树结点
struct Node {
    int val;
    Node * parent;
    Node * left;
    Node * right;
    int size;
    int height;

    Node(int val) {
        this->val = val;
        this->parent = nullptr;
        this->left = nullptr;
        this->right = nullptr;
        this->height = 0; // 结点高度：以node为根节点的子树的高度（高度定义：叶结点的高度是0）
        this->size = 1; // 结点元素数：以node为根节点的子树的节点总数
    }

    Node(int val, Node * parent) {
        this->val = val;
        this->parent = parent;
        this->left = nullptr;
        this->right = nullptr;
        this->height = 0; // 结点高度：以node为根节点的子树的高度（高度定义：叶结点的高度是0）
        this->size = 1; // 结点元素数：以node为根节点的子树的节点总数
    }

    Node(int val, Node * parent, Node * left, Node * right) {
        this->val = val;
        this->parent = parent;
        this->left = left;
        this->right = right;
        this->height = 0; // 结点高度：以node为根节点的子树的高度（高度定义：叶结点的高度是0）
        this->size = 1; // 结点元素数：以node为根节点的子树的节点总数
    }
};


// 平衡二叉搜索树（AVL树）：允许重复值
class AVL {
public:
    AVL(vector<int> & vals) {
        if (!vals.empty()) {
            root = build(vals, 0, vals.size() - 1, nullptr);
        }
    }

    // 根据vals[l:r]构造平衡二叉搜索树 -> 返回根结点
    Node * build(vector<int> & vals, int l, int r, Node * parent) {
        int m = (l + r) >> 1;
        Node * node = new Node(vals[m], parent);
        if (l <= m - 1) {
            node->left = build(vals, l, m - 1, node);
        }
        if (m + 1 <= r) {
            node->right = build(vals, m + 1, r, node);
        }
        recompute(node);
        return node;
    }

    // 返回二叉搜索树中第k小的元素
    int kthSmallest(int k) {
        Node * node = root;
        while (node != nullptr) {
            int left = getSize(node->left);
            if (left < k - 1) {
                node = node->right;
                k -= left + 1;
            } else if (left == k - 1) {
                break;
            } else {
                node = node->left;
            }
        }
        return node->val;
    }

    void insert(int v) {
        if (root == nullptr) {
            root = new Node(v);
        } else {
            // 计算新结点的添加位置
            Node * node = subtreeSearch(root, v);
            bool isAddLeft = v <= node->val; // 是否将新结点添加到node的左子结点
            if (node->val == v) { // 如果值为v的结点已存在
                if (node->left != nullptr) { // 值为v的结点存在左子结点，则添加到其左子树的最右侧
                    node = subtreeLast(node->left);
                    isAddLeft = false;
                } else { // 值为v的结点不存在左子结点，则添加到其左子结点
                    isAddLeft = true;
                }
            }

            // 添加新结点
            Node * leaf = new Node(v, node);
            if (isAddLeft) {
                node->left = leaf;
            } else {
                node->right = leaf;
            }

            rebalance(leaf);
        }
    }

    // 删除值为v的结点 -> 返回是否成功删除结点
    bool Delete(int v) {
        if (root == nullptr) {
            return false;
        }

        Node * node = subtreeSearch(root, v);
        if (node->val != v) { // 没有找到需要删除的结点
            return false;
        }

        // 处理当前结点既有左子树也有右子树的情况
        // 若左子树比右子树高度低，则将当前结点替换为右子树最左侧的结点，并移除右子树最左侧的结点
        // 若右子树比左子树高度低，则将当前结点替换为左子树最右侧的结点，并移除左子树最右侧的结点
        if (node->left != nullptr && node->right != nullptr) {
            Node * replacement = nullptr;
            if (node->left->height <= node->right->height) {
                replacement = subtreeFirst(node->right);
            } else {
                replacement = subtreeLast(node->left);
            }
            node->val = replacement->val;
            node = replacement;
        }

        Node * parent = node->parent;
        Delete(node);
        rebalance(parent);
        return true;
    }

private:
    Node * root;

    // 删除结点p并用它的子结点代替它，结点p至多只能有1个子结点
    void Delete(Node * node) {
        if (node->left != nullptr && node->right != nullptr) {
            return;
            // throw new Exception("Node has two children");
        }
        Node * child = node->left != nullptr ? node->left : node->right;
        if (child != nullptr) {
            child->parent = node->parent;
        }
        if (node == root) {
            root = child;
        } else {
            Node * parent = node->parent;
            if (node == parent->left) {
                parent->left = child;
            } else {
                parent->right = child;
            }
        }
        node->parent = node;
    }

    // 在以node为根结点的子树中搜索值为v的结点，如果没有值为v的结点，则返回值为v的结点应该在的位置的父结点
    Node * subtreeSearch(Node * node, int v) {
        if (node->val < v && node->right != nullptr) {
            return subtreeSearch(node->right, v);
        } else if (node->val > v && node->left != nullptr) {
            return subtreeSearch(node->left, v);
        } else {
            return node;
        }
    }

    // 重新计算node结点的高度和元素数
    void recompute(Node * node) {
        node->height = 1 + max(getHeight(node->left), getHeight(node->right));
        node->size = 1 + getSize(node->left) + getSize(node->right);
    }

    // 从node结点开始（含node结点）逐个向上重新平衡二叉树，并更新结点高度和元素数
    void rebalance(Node * node) {
        while (node != nullptr) {
            int oldHeight = node->height, oldSize = node->size;
            if (!isBalanced(node)) {
                node = restructure(tallGrandchild(node));
                recompute(node->left);
                recompute(node->right);
            }
            recompute(node);
            if (node->height == oldHeight && node->size == oldSize) {
                node = nullptr; // 如果结点高度和元素数都没有变化则不需要再继续向上调整
            } else {
                node = node->parent;
            }
        }
    }

    // 判断node结点是否平衡
    bool isBalanced(Node * node) {
        return abs(getHeight(node->left) - getHeight(node->right)) <= 1;
    }

    // 获取node结点更高的子树
    Node * tallChild(Node * node) {
        if (getHeight(node->left) > getHeight(node->right)) {
            return node->left;
        } else {
            return node->right;
        }
    }

    // 获取node结点更高的子树中的更高的子树
    Node * tallGrandchild(Node * node) {
        Node * child = tallChild(node);
        return tallChild(child);
    }

    // 重新连接父结点和子结点（子结点允许为空）
    static void relink(Node * parent, Node * child, bool isLeft) {
        if (isLeft) {
            parent->left = child;
        } else {
            parent->right = child;
        }
        if (child != nullptr) {
            child->parent = parent;
        }
    }

    // 旋转操作
    void rotate(Node * node) {
        Node * parent = node->parent;
        Node * grandparent = parent->parent;
        if (grandparent == nullptr) {
            root = node;
            node->parent = nullptr;
        } else {
            relink(grandparent, node, parent == grandparent->left);
        }

        if (node == parent->left) {
            relink(parent, node->right, true);
            relink(node, parent, false);
        } else {
            relink(parent, node->left, false);
            relink(node, parent, true);
        }
    }

    // trinode操作
    Node * restructure(Node * node) {
        Node * parent = node->parent;
        Node * grandparent = parent->parent;

        if ((node == parent->right) == (parent == grandparent->right)) { // 处理需要一次旋转的情况
            rotate(parent);
            return parent;
        } else { // 处理需要两次旋转的情况：第1次旋转后即成为需要一次旋转的情况
            rotate(node);
            rotate(node);
            return node;
        }
    }

    // 返回以node为根结点的子树的第1个元素
    static Node * subtreeFirst(Node * node) {
        while (node->left != nullptr) {
            node = node->left;
        }
        return node;
    }

    // 返回以node为根结点的子树的最后1个元素
    static Node * subtreeLast(Node * node) {
        while (node->right != nullptr) {
            node = node->right;
        }
        return node;
    }

    // 获取以node为根结点的子树的高度
    static int getHeight(Node * node) {
        return node != nullptr ? node->height : 0;
    }

    // 获取以node为根结点的子树的结点数
    static int getSize(Node * node) {
        return node != nullptr ? node->size : 0;
    }
};

class Solution {
public:
    int kthSmallest(TreeNode * root, int k) {
        // 中序遍历生成数值列表
        vector<int> inorderList;
        inorder(root, inorderList);
        // 构造平衡二叉搜索树
        AVL avl(inorderList);

        // 模拟1000次插入和删除操作
        vector<int> randomNums(1000);
        std::random_device rd;
        for (int i = 0; i < 1000; ++i) {
            randomNums[i] = rd()%(10001);
            avl.insert(randomNums[i]);
        }
        shuffle(randomNums); // 列表乱序
        for (int i = 0; i < 1000; ++i) {
            avl.Delete(randomNums[i]);
        }

        return avl.kthSmallest(k);
    }

private:
    void inorder(TreeNode * node, vector<int> & inorderList) {
        if (node->left != nullptr) {
            inorder(node->left, inorderList);
        }
        inorderList.push_back(node->val);
        if (node->right != nullptr) {
            inorder(node->right, inorderList);
        }
    }

    void shuffle(vector<int> & arr) {
        std::random_device rd;
        int length = arr.size();
        for (int i = 0; i < length; i++) {
            int randIndex = rd()%length;
            swap(arr[i],arr[randIndex]);
        }
    }
};
```





## 53.二叉树的右视图

给定一个二叉树的 **根节点** `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值。

 

**示例 1:**

![img](https://assets.leetcode.com/uploads/2021/02/14/tree.jpg)

```
输入: [1,2,3,null,5,null,4]
输出: [1,3,4]
```

**示例 2:**

```
输入: [1,null,3]
输出: [1,3]
```

**示例 3:**

```
输入: []
输出: []
```

 

**提示:**

- 二叉树的节点个数的范围是 `[0,100]`
- `-100 <= Node.val <= 100` 



### 思路

BFS层序遍历二叉树，每次将该层最后一个节点压入`ans`即可

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        if (!root) return {};
        vector<int> res;
        deque<TreeNode*> deq;
        deq.push_back(root);
        while (!deq.empty()) {
            int sz = deq.size();
            int num = deq.back()->val;
            res.push_back(num);
            for (int i = 0; i < sz; ++i) {
                auto p = deq.front();
                deq.pop_front();
                if (p->left) deq.push_back(p->left);
                if (p->right) deq.push_back(p->right);
            }
        }
        return res;
    }
};
```





## 54.二叉树展开为链表

给你二叉树的根结点 `root` ，请你将它展开为一个单链表：

- 展开后的单链表应该同样使用 `TreeNode` ，其中 `right` 子指针指向链表中下一个结点，而左子指针始终为 `null` 。
- 展开后的单链表应该与二叉树 [**先序遍历**](https://baike.baidu.com/item/先序遍历/6442839?fr=aladdin) 顺序相同。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/01/14/flaten.jpg)

```
输入：root = [1,2,5,3,4,null,6]
输出：[1,null,2,null,3,null,4,null,5,null,6]
```

**示例 2：**

```
输入：root = []
输出：[]
```

**示例 3：**

```
输入：root = [0]
输出：[0]
```

 

**提示：**

- 树中结点数在范围 `[0, 2000]` 内
- `-100 <= Node.val <= 100`



### 思路

本题二叉树展开为链表的顺序就是二叉树的前序遍历顺序。所以我们只需要前序遍历一次二叉树，将节点存在vector中再进行调整就好了

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    void preorderTraversal(TreeNode* root, vector<TreeNode*> &l) {
        if (root != nullptr) {
            l.push_back(root);
            preorderTraversal(root->left, l);
            preorderTraversal(root->right, l);
        }
    }
    
    void flatten(TreeNode* root) {
        vector<TreeNode*> l;
        preorderTraversal(root, l);
        int n = l.size();
        for (int i = 1; i < n; ++i) {
            TreeNode *prev = l.at(i - 1), *curr = l.at(i);
            prev->left = nullptr;
            prev->right = curr;
        }
    }
};
```

上面的方法用了额外的空间，我们可以在原地调整二叉树来省下这个空间。

采用头插法构建链表，以示例 1 为例，也就是从节点 6 开始，在 6 的前面插入 5，在 5 的前面插入 4，依此类推。

为此，我们需要按照 `6->5->4->3->2->1` 的顺序访问节点，也就是按照**右子树 - 左子树 - 根**的顺序 DFS 这棵树。

DFS 的同时，记录当前链表的头节点 head，一开始 head 为空节点。

具体来说：

1. 如果当前节点为空，返回
2. 递归右子树
3. 递归左子树
4. 把 `root->left` 置空
5. 头插法，把 `root` 插到 `head` 前面，也就是 `root->right = head`
6. 现在 `root` 是链表的头节点，把 `head` 更新为 `root`

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
private:
    TreeNode* head;
public:
    void flatten(TreeNode* root) {
        if (!root) return;
        flatten(root->right);
        flatten(root->left);
        root->left = nullptr;
        root->right = head;
        head = root;
    }
};
```





## 55.从前序与中序遍历序列构造二叉树

给定两个整数数组 `preorder` 和 `inorder` ，其中 `preorder` 是二叉树的**先序遍历**， `inorder` 是同一棵树的**中序遍历**，请构造二叉树并返回其根节点。

 

**示例 1:**

![img](https://assets.leetcode.com/uploads/2021/02/19/tree.jpg)

```
输入: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
输出: [3,9,20,null,null,15,7]
```

**示例 2:**

```
输入: preorder = [-1], inorder = [-1]
输出: [-1]
```

 

**提示:**

- `1 <= preorder.length <= 3000`
- `inorder.length == preorder.length`
- `-3000 <= preorder[i], inorder[i] <= 3000`
- `preorder` 和 `inorder` 均 **无重复** 元素
- `inorder` 均出现在 `preorder`
- `preorder` **保证** 为二叉树的前序遍历序列
- `inorder` **保证** 为二叉树的中序遍历序列



### 思路

从前序遍历中可以得到哪个节点是根节点，从中序遍历中可以得到当前根节点的左右子树的节点数目。

先得到**左子树的节点数目`size_left_subtree`**，之后根据**先序遍历中「从左边界+1开始的size_left_subtree」个元素就对应了中序遍历中「从左边界开始到根节点定位-1」的元素**，递归构造左子树，并连接到根节点，然后根据**先序遍历中「从左边界+1+左子树节点数目开始到右边界」的元素就对应了中序遍历中「从根节点定位+1到右边界」的元素**，递归构造右子树，并连接到根节点，最后返回根节点即可。对于根据前序遍历的元素找到中序遍历的下标，可以通过哈希表实现，让查找元素的时间复杂度为`O(1)`。

```c++
class Solution {
private:
    unordered_map<int, int> index;

public:
    TreeNode* myBuildTree(const vector<int>& preorder, const vector<int>& inorder, int preorder_left, int preorder_right, int inorder_left, int inorder_right) {
        // preorder_left和preorder_right分别表示先序遍历的左右边界
        // inorder_left和inorder_right分别表示中序遍历的左右边界
        if (preorder_left > preorder_right) {
            return nullptr;
        }
        
        // 前序遍历中的第一个节点就是根节点
        int preorder_root = preorder_left;
        // 在中序遍历中定位根节点
        int inorder_root = index[preorder[preorder_root]];
        
        // 先把根节点建立出来
        TreeNode* root = new TreeNode(preorder[preorder_root]);
        // 得到左子树中的节点数目
        int size_left_subtree = inorder_root - inorder_left;
        // 递归地构造左子树，并连接到根节点
        // 先序遍历中「从左边界+1开始的size_left_subtree」个元素就对应了中序遍历中「从左边界开始到根节点定位-1」的元素
        root->left = myBuildTree(preorder, inorder, preorder_left + 1, preorder_left + size_left_subtree, inorder_left, inorder_root - 1);
        // 递归地构造右子树，并连接到根节点
        // 先序遍历中「从左边界+1+左子树节点数目开始到右边界」的元素就对应了中序遍历中「从根节点定位+1到右边界」的元素
        root->right = myBuildTree(preorder, inorder, preorder_left + size_left_subtree + 1, preorder_right, inorder_root + 1, inorder_right);
        return root;
    }

    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        int n = preorder.size();
        // 构造哈希映射，帮助我们快速定位根节点
        for (int i = 0; i < n; ++i) {
            index[inorder[i]] = i;
        }
        return myBuildTree(preorder, inorder, 0, n - 1, 0, n - 1);
    }
};
```

另一种比较直白的写法（没有用到哈希表）

需要注意：在C++中，使用两个迭代器来构造一个 `std::vector` 或其他容器时，指定的范围是**左闭右开区间**，即 `[start_it, end_it)`

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if (preorder.empty()) return nullptr;
        int left_sz = ranges::find(inorder, preorder[0]) - inorder.begin();
        // 左子树前序遍历
        vector<int> pre1 = vector<int>(preorder.begin() + 1, preorder.begin() + 1 + left_sz);
        // 右子树前序遍历
        vector<int> pre2 = vector<int>(preorder.begin() + 1 + left_sz, preorder.end());
        // 左子树中序遍历
        vector<int> in1 = vector<int>(inorder.begin(), inorder.begin() + left_sz);
        // 右子树中序遍历
        vector<int> in2 = vector<int>(inorder.begin() + left_sz + 1, inorder.end());
        TreeNode* left = buildTree(pre1, in1);
        TreeNode* right = buildTree(pre2, in2);
        return new TreeNode(preorder[0], left, right);
    }
};
```



## 56.路径总和Ⅲ

给定一个二叉树的根节点 `root` ，和一个整数 `targetSum` ，求该二叉树里节点值之和等于 `targetSum` 的 **路径** 的数目。

**路径** 不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/04/09/pathsum3-1-tree.jpg)

```
输入：root = [10,5,-3,3,2,null,11,3,-2,null,1], targetSum = 8
输出：3
解释：和等于 8 的路径有 3 条，如图所示。
```

**示例 2：**

```
输入：root = [5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22
输出：3
```

 

**提示:**

- 二叉树的节点个数的范围是 `[0,1000]`
- `-10^9 <= Node.val <= 10^9` 
- `-1000 <= targetSum <= 1000` 



### 思路

可以直接深搜，**我们定义`rootSum(p, val)`表示以节点`p`为起点向下且满足路径总和为`val`的路径数目**。我们对二叉树上每个节点`p`求出`root(p, targetSum)`然后对这些路径数目求和为返回结果

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int pathSum(TreeNode* root, int targetSum) {
        if (!root) return 0;
        // rootSum(p, val) 表示以 root 为起点向下且满足路经总和为 val 的路径数目
        auto rootSum = [&](auto&& rootSum, TreeNode* root, long long targetSum) -> int {
            if (!root) return 0;
            int res = 0;
            if (root->val == targetSum) ++res;
            res += rootSum(rootSum, root->left, targetSum - root->val);
            res += rootSum(rootSum, root->right, targetSum - root->val);
            return res;
        };
        int ans = rootSum(rootSum, root, targetSum);
        ans += pathSum(root->left, targetSum);
        ans += pathSum(root->right, targetSum);
        return ans;
    }
};
```





## 57.二叉树的最近公共祖先

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

[百度百科](https://baike.baidu.com/item/最近公共祖先/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2018/12/14/binarytree.png)

```
输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
输出：3
解释：节点 5 和节点 1 的最近公共祖先是节点 3 。
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2018/12/14/binarytree.png)

```
输入：root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
输出：5
解释：节点 5 和节点 4 的最近公共祖先是节点 5 。因为根据定义最近公共祖先节点可以为节点本身。
```

**示例 3：**

```
输入：root = [1,2], p = 1, q = 2
输出：1
```

 

**提示：**

- 树中节点数目在范围 `[2, 10^5]` 内。
- `-10^9 <= Node.val <= 10^9`
- 所有 `Node.val` `互不相同` 。
- `p != q`
- `p` 和 `q` 均存在于给定的二叉树中。



### 思路

这是一个高频面试题，基本思路是用一个哈希表存放每个节点的父节点，对整棵二叉树`dfs`一遍来创建哈希表。**然后再用另一个哈希表存放父节点是否访问过**。之后从目标`p`和`q`节点开始网上找最近公共祖先。

先从`p`开始一直往上找，直到整棵树的根节点为止，每次将遍历到的节点记作`true`已访问。

然后从`q`开始往上找，直到找到第一个标记为`true`的节点，即为最近公共祖先。

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
private:
    unordered_map<TreeNode*, TreeNode*> fa;
    unordered_map<TreeNode*, bool> vis;
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        auto dfs = [&](auto&& dfs, TreeNode* root, TreeNode* pre) {
            if (!root) return;
            fa[root] = pre;
            vis[root] = false;
            dfs(dfs, root->left, root);
            dfs(dfs, root->right, root);
        };
        dfs(dfs, root, nullptr);
        while (p) {
            vis[p] = true;
            p = fa[p];
        }
        while (q) {
            if (vis[q]) return q;
            q = fa[q];
        }
        return nullptr;
    }
};
```





## 58.二叉树中的最大路径和（树形dp）

二叉树中的 **路径** 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。

**路径和** 是路径中各节点值的总和。

给你一个二叉树的根节点 `root` ，返回其 **最大路径和** 。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/13/exx1.jpg)

```
输入：root = [1,2,3]
输出：6
解释：最优路径是 2 -> 1 -> 3 ，路径和为 2 + 1 + 3 = 6
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/10/13/exx2.jpg)

```
输入：root = [-10,9,20,null,null,15,7]
输出：42
解释：最优路径是 15 -> 20 -> 7 ，路径和为 15 + 20 + 7 = 42
```

 

**提示：**

- 树中节点数目范围是 `[1, 3 * 10^4]`
- `-1000 <= Node.val <= 1000`



### 思路

本题求二叉树中的最大路径和，**就是求以每个节点为根节点的最大路径和**。我们以每个节点的角度来看，**假设当前节点为`node`，以`node`为根节点所能得到的最大路径和等于`max(左子树的最大路径和, 右子树的最大路径和) + node->val`**。

这就可以看出这是一道树形 dp 问题，我们要求的当前最大路径和的子问题就是节点的左右子树的最大路径和。

所以我们枚举每个`node`，递归的计算`node`左右子树的最大路径和再加上`node->val`即可。

**需要注意的是，如果递归计算的过程中出现左右子树的最大路径和为负数的情况，需要舍弃（不选这个子树），返回0即可**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int maxPathSum(TreeNode* root) {
        int res = INT_MIN;
        // 计算以 root 为根，往下递归的最大路径和
        auto dfs = [&](auto&& dfs, TreeNode* root) -> int{
            if (!root) return 0;
            int left_val = dfs(dfs, root->left);
            int right_val = dfs(dfs, root->right);
            res = max(res, left_val + right_val + root->val);
            return max(max(left_val, right_val) + root->val, 0);
        };
        dfs(dfs, root);
        return res;
    }
};
```





## 59.爬楼梯

假设你正在爬楼梯。需要 `n` 阶你才能到达楼顶。

每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶呢？

 

**示例 1：**

```
输入：n = 2
输出：2
解释：有两种方法可以爬到楼顶。
1. 1 阶 + 1 阶
2. 2 阶
```

**示例 2：**

```
输入：n = 3
输出：3
解释：有三种方法可以爬到楼顶。
1. 1 阶 + 1 阶 + 1 阶
2. 1 阶 + 2 阶
3. 2 阶 + 1 阶
```

 

**提示：**

- `1 <= n <= 45`



### 思路

简单dp即可，注意数据范围开long long

```c++
class Solution {
public:
    long long climbStairs(int n) {
        long long dp[n+1];
        dp[0] = 1, dp[1] = 2;
        for (int i = 2; i <= n; ++i) dp[i] = dp[i-1] + dp[i-2];
        return dp[n-1];
    }
};
```





## 60.杨辉三角

给定一个非负整数 *`numRows`，*生成「杨辉三角」的前 *`numRows`* 行。

在「杨辉三角」中，每个数是它左上方和右上方的数的和。

 

**示例 1:**

```
输入: numRows = 5
输出: [[1],[1,1],[1,2,1],[1,3,3,1],[1,4,6,4,1]]
```

**示例 2:**

```
输入: numRows = 1
输出: [[1]]
```

 

**提示:**

- `1 <= numRows <= 30`



### 思路

简单dp

```c++
class Solution {
public:
    vector<vector<int>> generate(int numRows) {
        vector<vector<int>> ans;
        ans.push_back({1});
        for (int i = 2; i <= numRows; ++i) {
            vector<int> line;
            for (int j = 0; j < i; ++j) {
                if (j == 0 || j == i - 1) line.push_back(1);
                else {
                    int temp = ans[i-2][j-1] + ans[i-2][j];
                    line.push_back(temp);
                }
            }
            ans.push_back(line);
        }
        return ans;
    }
};
```





## 61.打家劫舍

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，**如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警**。

给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。

 

**示例 1：**

```
输入：[1,2,3,1]
输出：4
解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。
```

**示例 2：**

```
输入：[2,7,9,3,1]
输出：12
解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
     偷窃到的最高金额 = 2 + 9 + 1 = 12 。
```

 

**提示：**

- `1 <= nums.length <= 100`
- `0 <= nums[i] <= 400`



### 思路

经典dp模板题

```c++
class Solution {
public:
    int rob(vector<int>& nums) {
        int len = nums.size();
        if (len == 1) return nums[0];
        int dp[105];
        dp[0] = nums[0];
        dp[1] = max(nums[0], nums[1]);
        for (int i = 2; i < len; ++i) dp[i] = max(dp[i-2] + nums[i], dp[i-1]);
        return dp[len-1];
    }
};
```





## 62.完全平方数

给你一个整数 `n` ，返回 *和为 `n` 的完全平方数的最少数量* 。

**完全平方数** 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，`1`、`4`、`9` 和 `16` 都是完全平方数，而 `3` 和 `11` 不是。

 

**示例 1：**

```
输入：n = 12
输出：3 
解释：12 = 4 + 4 + 4
```

**示例 2：**

```
输入：n = 13
输出：2
解释：13 = 4 + 9
```

 

**提示：**

- `1 <= n <= 10^4`



### 思路

定义 `f[i]` 表示最少需要多少个数的平方表示整数 `i` ，我们从 1 开始枚举到 n 。对于每个枚举到的 `i` ，再从 1 开始枚举构成 `i` 的最少数量。假设当前枚举到 `j` ，那么还需要取若干数的平方，构成 `i - j^2` ，于是构成整数 `i` 的最小平方数个数就是 `f[i-j*j] + 1` 。

```c++
class Solution {
public:
    int numSquares(int n) {
        // f[i] 表示最少需要多少个数表示整数 i
        vector<int> f(n + 1);
        for (int i = 1; i <= n; ++i) {
            // 记录表示整数 i 所需要的最少数的个数
            int minn = INT_MAX;
            for (int j = 1; j * j <= i; ++j) {
                minn = min(minn, f[i-j*j]);
            }
            f[i] = minn + 1;
        }
        return f[n];
};
```





## 63.零钱兑换

给你一个整数数组 `coins` ，表示不同面额的硬币；以及一个整数 `amount` ，表示总金额。

计算并返回可以凑成总金额所需的 **最少的硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `-1` 。

你可以认为每种硬币的数量是无限的。

 

**示例 1：**

```
输入：coins = [1, 2, 5], amount = 11
输出：3 
解释：11 = 5 + 5 + 1
```

**示例 2：**

```
输入：coins = [2], amount = 3
输出：-1
```

**示例 3：**

```
输入：coins = [1], amount = 0
输出：0
```

 

**提示：**

- `1 <= coins.length <= 12`
- `1 <= coins[i] <= 2^31 - 1`
- `0 <= amount <= 10^4`



### 思路

典型的dp，我们定义`dp[i]`为**凑满`i`元所需要的最少硬币个数**，那么`dp[i]`就依赖于前面的子问题`dp[i-coins[j]]`，

即在`dp[i-coins[j]]`的基础上再加上一个硬币就是**凑满`i`元所需要的最少硬币个数**。

注意在遍历的过程中还要判断`i`和`coins[j]`的大小关系

```c++
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        int len = coins.size();
        vector<int> dp(amount+1, 10005); // dp[i]表示i元最少需要多少个硬币
        dp[0] = 0;
        for (int i = 1; i <= amount; ++i) {
            for (int j = 0; j < coins.size(); ++j) {
                if (i >= coins[j]) dp[i] = min(dp[i], dp[i-coins[j]] + 1);
            }
        }
        return dp[amount] < 10005 ? dp[amount] : -1;
    }
};
```





## 64.单词拆分（字符串dp）

给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

**注意：**不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

 

**示例 1：**

```
输入: s = "leetcode", wordDict = ["leet", "code"]
输出: true
解释: 返回 true 因为 "leetcode" 可以由 "leet" 和 "code" 拼接成。
```

**示例 2：**

```
输入: s = "applepenapple", wordDict = ["apple", "pen"]
输出: true
解释: 返回 true 因为 "applepenapple" 可以由 "apple" "pen" "apple" 拼接成。
     注意，你可以重复使用字典中的单词。
```

**示例 3：**

```
输入: s = "catsandog", wordDict = ["cats", "dog", "sand", "and", "cat"]
输出: false
```

 

**提示：**

- `1 <= s.length <= 300`
- `1 <= wordDict.length <= 1000`
- `1 <= wordDict[i].length <= 20`
- `s` 和 `wordDict[i]` 仅由小写英文字母组成
- `wordDict` 中的所有字符串 **互不相同**



### 思路

我们分析原问题的子问题：想要判断`s[0..i]`这个字符串是否合法，就要判断其子串`s[0..j-1]`是否合法，然后再判断`s[j..i-1]`是否在给出的`wordDict`中。所以我们可以分别枚举`s[0..i]`和`s[0..j-1]`，在枚举的过程中判断`s[j..i-1]`是否在`wordDict`中

```c++
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        auto wordDictSet = unordered_set<string>();

        // 去重，set方便查找
        for (auto word : wordDict) wordDictSet.insert(word);

        // dp[i]表示从0到第i个字符的字符串是否合法
        auto dp = vector<bool>(s.size() + 1);
        dp[0] = true; // 空字符串合法
        for (int i = 1; i <= s.size(); ++i) {
            for (int j = 0; j < i; ++j) {
                // 如果0..j合法，且j..i-j合法，那么0..i就合法
                if (dp[j] 
                && wordDictSet.find(s.substr(j, i-j)) != wordDictSet.end()) {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.size()];
    }
};
```





## 65.最长递增子序列

给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。

**子序列** 是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。





**示例 1：**

```
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
```

**示例 2：**

```
输入：nums = [0,1,0,3,2,3]
输出：4
```

**示例 3：**

```
输入：nums = [7,7,7,7,7,7,7]
输出：1
```

 

**提示：**

- `1 <= nums.length <= 2500`
- `-10^4 <= nums[i] <= 10^4`



### 思路

对于数组中的每个数字，**我们定义`dp[i]`为以第`i`个数结尾的最长递增子序列的长度**

于是原问题可以转换为求`max(dp)`，而`dp[i]`又依赖于前面的`dp[j]`，其中`i > j`

如果满足`nums[i] > nums[j]`，那么当前数字可以接到`dp[j]`之后。

然后还要对`dp[i]`和`dp[j] + 1`取一个`max`

```c++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int len = nums.size();
        vector<int> dp(len, 1); // dp[i]表示以第i个数结尾的最长递增子序列
        int ans = 1;
        dp[0] = 1;
        for (int i = 1; i < len; ++i) {
            for (int j = 0; j < i; ++j) {
                if (nums[i] > nums[j]) {
                    dp[i] = max(dp[j] + 1, dp[i]);
                    ans = ans > dp[i] ? ans:dp[i];
                }
            }
        }
        return ans;
    }
};
```





## 66.乘积最大子数组

给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。



测试用例的答案是一个 **32-位** 整数。

 

**示例 1:**

```
输入: nums = [2,3,-2,4]
输出: 6
解释: 子数组 [2,3] 有最大乘积 6。
```

**示例 2:**

```
输入: nums = [-2,0,-1]
输出: 0
解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。
```

 

**提示:**

- `1 <= nums.length <= 2 * 104`
- `-10 <= nums[i] <= 10`
- `nums` 的任何前缀或后缀的乘积都 **保证** 是一个 **32-位** 整数



### 思路

由于是乘积，可能出现本来是最大的数乘上负数后变成了最小的数，因此我们需要记录最大和最小的数。

定义：`f_max[i]` 表示以 `nums[i]` 结尾的最大乘积，`f_min[i]` 表示以 `nums[i]` 结尾的最小乘积。

在状态转移时，需要在 `f_max[i-1] * nums[i]、f_min[i-1] * nums[i] 和 nums[i]` 中选出最大值与最小值，最后返回 `f_max` 数组中的最大值即可。

```c++
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int n = nums.size();
        vector<int> f_max(n, 0);
        vector<int> f_min(n, 0);
        f_max[0] = f_min[0] = nums[0];
        for (int i = 1; i < n; ++i) {
            f_max[i] = max({f_max[i-1] * nums[i], f_min[i-1] * nums[i], nums[i]});
            f_min[i] = min({f_max[i-1] * nums[i], f_min[i-1] * nums[i], nums[i]});
        }
        return *max_element(f_max.begin(), f_max.end());
    }
};
```





## 67.分割等和子集（01背包）

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



### 思路

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



### 一维数组优化

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



## 68.最长有效括号

给你一个只包含 `'('` 和 `')'` 的字符串，找出最长有效（格式正确且连续）括号子串的长度。



 

**示例 1：**

```
输入：s = "(()"
输出：2
解释：最长有效括号子串是 "()"
```

**示例 2：**

```
输入：s = ")()())"
输出：4
解释：最长有效括号子串是 "()()"
```

**示例 3：**

```
输入：s = ""
输出：0
```

 

**提示：**

- `0 <= s.length <= 3 * 10^4`
- `s[i]` 为 `'('` 或 `')'`



### 思路1：动态规划

**有效括号子串是指在当前子串中，所有的左括号都正确的与右括号相匹配，**我们要求的是这种子串的最长长度。

定义`dp[i]`表示**字符串中第`i`个字符结尾的最长子串长度**

- 对于`'('`，以它结尾不可能构成满足条件的子串，所以对于所有的左括号将`dp`值初始化为0。
- 对于`')'`，假设这个右括号的下标为`i`，那么我们需要考虑第`i-1`个括号
  - 如果是`'('`，将其与当前的`'('`匹配，即`dp[i] = 2`。同时，**还需要考虑`i-1`之前的括号是否满足，即当`i-2 >= 0`时，`dp[i] = dp[i] + dp[i-2]`**，因为可能存在`'()()'`这种情况。
  - 如果是`')'`，就直接考虑`dp[i-1]`。然后找到当前`')'`的对称位置：`i - dp[i-1] - 1`，如果`i - dp[i-1] - 1 >= 0 && s[i-dp[i-1]-1] == '('`，**那么就在以第`i-1`个字符结尾的子串前面有一个与第`i`位右括号匹配的左括号（仔细理解这句话）**。所以`dp[i] = dp[i-1] + 2`。与上一种情况类似，还需要考虑`i - dp[i-1] - 1`之前的子串是否满足要求，所以有当`i - dp[i-1] - 2 >= 0`时，`dp[i] + dp[i-dp[i-1]-2]`。

最后返回dp数组的最大值即可

```c++
class Solution {
public:
    int longestValidParentheses(string s) {
        int size = s.length();
        vector<int> dp(size, 0);

        int maxVal = 0;
        for (int i = 1; i < size; ++i) {
            if (s[i] == ')') {
                if (s[i-1] == '(') {
                    dp[i] = 2;
                    if (i - 2 >= 0) dp[i] = dp[i] + dp[i-2];
                } else if (dp[i-1] > 0) {
                    if (i - dp[i-1] - 1 >= 0 && s[i-dp[i-1]-1] == '(') {
                        dp[i] = dp[i-1] + 2;
                        if ((i - dp[i-1] - 2) >= 0) dp[i] = dp[i] + dp[i-dp[i-1]-2];
                    }
                }
            }
            maxVal = max(maxVal, dp[i]);
        }
        return maxVal;
    }
};
```

### 思路2：栈

我们始终保持栈底元素为当前已经遍历过的元素中**最后一个没有被匹配的右括号的下标**，这样的做法主要是考虑了边界条件的处理，栈里其他元素维护左括号的下标：

- 对于遇到的 `'('` ，我们将它的下标放入栈中
- 对于遇到的 `')'` ，我们先弹出栈顶元素表示匹配了当前右括号，接下来判断：
  - 如果栈空，说明**当前的右括号为没有被匹配的右括号**，接下来把他的下标放入栈中来更新之前提到的**最后一个没有被匹配的右括号的下标**
  - 如果栈非空，当前右括号的下标减去栈顶元素即为**以该右括号为结尾的最长有效括号的长度**

从前往后遍历字符串并更新答案即可。

如果一开始栈空，并且第一个字符为左括号，那就无法满足“栈底元素始终是右括号的下标”这一条件。于是首先往栈中压入 -1。

```c++
#include<bits/stdc++.h>
using namespace std;

/*
栈如果非空，栈底永远存的是当前遍历过的字符串中上一个没有被匹配的右括号的下标
上一个没有被匹配的右括号的下标可以理解为每段有效括号之间的 “隔板”
比如，())((()))，第三个右括号，即左右 2 段匹配括号中间的 “隔板”
“隔板” 的存在影响计算最长括号长度，如果不存在 “隔板”，前后 2 段括号应该 “融合” 在一起，最长长度为：2 + 6 = 8
但此处因为 “隔板” 存在，所以最长长度只能为 6 
*/
int solve(string s) {
    int len = s.length();
    stack<int> stk;
    stk.push(-1);
    int res = 0;
    for (int i = 0; i < len; ++i) {
        if (s[i] == '(') {
            stk.push(i);
        } else {
            stk.pop();
            if (stk.empty()) {
                stk.push(i);
            } else {
                res = max(res, i - stk.top());
            }
        }
    }
    return res;
}

int main() {
    cout << solve("(()") << endl;
    cout << solve(")()())") << endl;
    cout << solve("") << endl;
    return 0;
}
```



## 69.不同路径

一个机器人位于一个 `m x n` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

 

**示例 1：**

![img](https://pic.leetcode.cn/1697422740-adxmsI-image.png)

```
输入：m = 3, n = 7
输出：28
```

**示例 2：**

```
输入：m = 3, n = 2
输出：3
解释：
从左上角开始，总共有 3 条路径可以到达右下角。
1. 向右 -> 向下 -> 向下
2. 向下 -> 向下 -> 向右
3. 向下 -> 向右 -> 向下
```

**示例 3：**

```
输入：m = 7, n = 3
输出：28
```

**示例 4：**

```
输入：m = 3, n = 3
输出：6
```

 

**提示：**

- `1 <= m, n <= 100`
- 题目数据保证答案小于等于 `2 * 10^9`



### 思路

简单的多维动态规划

```c++
class Solution {
public:
    int uniquePaths(int m, int n) {
        int dp[105][105];
        for (int i = 0; i < m; ++i) dp[i][0] = 1;
        for (int i = 1; i < n; ++i) dp[0][i] = 1;

        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        return dp[m - 1][n - 1];
    }
};
```





## 70.最小路径和

给定一个包含非负整数的 `*m* x *n*` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

**说明：**每次只能向下或者向右移动一步。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/11/05/minpath.jpg)

```
输入：grid = [[1,3,1],[1,5,1],[4,2,1]]
输出：7
解释：因为路径 1→3→1→1→1 的总和最小。
```

**示例 2：**

```
输入：grid = [[1,2,3],[4,5,6]]
输出：12
```

 

**提示：**

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 200`
- `0 <= grid[i][j] <= 200`



### 思路

和上一题类似

```c++
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int dp[205][205];
        int m = grid.size();
        int n = grid[0].size();
        dp[0][0] = grid[0][0];
        for (int i = 1; i < n; ++i) dp[0][i] = grid[0][i] + dp[0][i - 1];
        for (int i = 1; i < m; ++i) dp[i][0] = grid[i][0] + dp[i - 1][0];

        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                dp[i][j] = grid[i][j] + min(dp[i - 1][j], dp[i][j - 1]);
            }
        }
        return dp[m - 1][n - 1];
    }
};
```





## 71.最长回文子串

给你一个字符串 `s`，找到 `s` 中最长的回文子串

**回文子串**：**子字符串是字符串中连续的非空字符序列**



 

**示例 1：**

```
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。
```

**示例 2：**

```
输入：s = "cbbd"
输出："bb"
```

 

**提示：**

- `1 <= s.length <= 1000`
- `s` 仅由数字和英文字母组成



### 思路

**定义`dp[i][j]`表示`s[i..j]`是否为回文串**，因为长度为1的子串都是回文串，所以先初始化`dp[i][i]`的值为1.

然后**开始枚举字串长度`L`**，在枚举长度的过程中枚举**左边界`i`**。设右边界为`j`，根据`L`和`i`和`j - i + 1 = L`得到`j = L + i - 1`，然后判断右边界是否越界，如果越界就退出此次左边界`i`的枚举。然后进行左右边界的判断

- 如果左右边界字母不相同，则`s[i..j]`就不是回文串
- 如果左右边界字母相同
  - 先判断`j - i + 1`是否小于4，如果比4小，**那么他们中间就只有1个字符或者没有字符，于是可以直接判断`s[i..j]`是回文串**
  - 如果比4大，那么就还要判断夹在他们中间的字符串是否为回文串，于是就有了状态转移方程：`dp[i][j] = dp[i+1][j-1]`

每次找到一个回文串就和当前最大长度比较，如果更大就更新，最后返回最大回文子串

```c++
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.size();
        if (n < 2) {
            return s;
        }

        int maxLen = 1;
        int begin = 0;
        // dp[i][j] 表示 s[i..j] 是否是回文串
        vector<vector<int>> dp(n, vector<int>(n));
        // 初始化：所有长度为 1 的子串都是回文串
        for (int i = 0; i < n; i++) {
            dp[i][i] = true;
        }
        // 递推开始
        // 先枚举子串长度
        for (int L = 2; L <= n; L++) {
            // 枚举左边界，左边界的上限设置可以宽松一些
            for (int i = 0; i < n; i++) {
                // 由 L 和 i 可以确定右边界，即 j - i + 1 = L 得
                int j = L + i - 1;
                // 如果右边界越界，就可以退出当前循环
                if (j >= n) {
                    break;
                }

                if (s[i] != s[j]) {
                    dp[i][j] = false;
                } else {
                    if (j - i < 3) {
                        dp[i][j] = true;
                    } else {
                        dp[i][j] = dp[i + 1][j - 1];
                    }
                }

                // 只要 dp[i][L] == true 成立，就表示子串 s[i..L] 是回文，此时记录回文长度和起始位置
                if (dp[i][j] && j - i + 1 > maxLen) {
                    maxLen = j - i + 1;
                    begin = i;
                }
            }
        }
        return s.substr(begin, maxLen);
    }
};
```





## 72.最长公共子序列

给定两个字符串 `text1` 和 `text2`，返回这两个字符串的最长 **公共子序列** 的长度。如果不存在 **公共子序列** ，返回 `0` 。

一个字符串的 **子序列** 是指这样一个新的字符串：它是由原字符串在不改变字符的相对顺序的情况下删除某些字符（也可以不删除任何字符）后组成的新字符串。

- 例如，`"ace"` 是 `"abcde"` 的子序列，但 `"aec"` 不是 `"abcde"` 的子序列。

两个字符串的 **公共子序列** 是这两个字符串所共同拥有的子序列。

 

**示例 1：**

```
输入：text1 = "abcde", text2 = "ace" 
输出：3  
解释：最长公共子序列是 "ace" ，它的长度为 3 。
```

**示例 2：**

```
输入：text1 = "abc", text2 = "abc"
输出：3
解释：最长公共子序列是 "abc" ，它的长度为 3 。
```

**示例 3：**

```
输入：text1 = "abc", text2 = "def"
输出：0
解释：两个字符串没有公共子序列，返回 0 。
```

 

**提示：**

- `1 <= text1.length, text2.length <= 1000`
- `text1` 和 `text2` 仅由小写英文字符组成。



### 思路

定义`dp[i][j]`表示`text1[0..i]`和`text2[0..j]`的最长公共子序列

两层循环遍历两个字符串

- 如果字符相等，就有`dp[i][j] = dp[i-1][j-1] + 1`
- 否则，`dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

```c++
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.length(), n = text2.length();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1));
        for (int i = 1; i <= m; ++i) {
            char c1 = text1.at(i-1);
            for (int j = 1; j <= n; ++j) {
                char c2 = text2.at(j-1);
                if (c1 == c2) dp[i][j] = dp[i-1][j-1] + 1;
                else dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
            }
        }
        return dp[m][n];
    }
};
```

至于为什么把 dp 数组的大小设置成 `(m + 1) * (n + 1)` ，这是为了避免出现负数下标，并且保证字符串的第一个字符也能够被全部计算到。

比如下面的错误代码，就没有保证字符串的第一个字符被全部计算到：

```c++
// 错误代码
#include<bits/stdc++.h>
using namespace std;

int solve(string text1, string text2) {
    int len1 = text1.length(), len2 = text2.length();
    vector<vector<int>> f(len1, vector<int>(len2));
    // 此处仅仅初始化了两个字符串的第一个字符匹配的情况
    if (text1[0] == text2[0]) f[0][0] = 1;
    else f[0][0] = 0;
    int res = f[0][0];
    // 实际上还需要计算 text1[0] 和 text2[0...j] 这种情况
    // 但是下面的循环是从 text1[1] 开始的，就说明上面仅仅考虑了 text1[0] 和 text2[0]
    for (int i = 1; i < len1; ++i) {
        for (int j = 1; j < len2; ++j) {
            if (text1[i] == text2[j]) {
                f[i][j] = f[i-1][j-1] + 1;
            } else {
                f[i][j] = max(f[i-1][j], f[i][j-1]);
            }
            res = max(f[i][j], res);
        }
    }
    return res;
}

int main() {
    string text1 = "abcde";
    string text2 = "ace";
    cout << solve(text1, text2) << endl;
    // 期望输出为 3，实际输出为 2
    return 0;
}
```



## 73.编辑距离

给你两个单词 `word1` 和 `word2`， *请返回将 `word1` 转换成 `word2` 所使用的最少操作数* 。

你可以对一个单词进行如下三种操作：

- 插入一个字符
- 删除一个字符
- 替换一个字符

 

**示例 1：**

```
输入：word1 = "horse", word2 = "ros"
输出：3
解释：
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
```

**示例 2：**

```
输入：word1 = "intention", word2 = "execution"
输出：5
解释：
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')
```

 

**提示：**

- `0 <= word1.length, word2.length <= 500`
- `word1` 和 `word2` 由小写英文字母组成



### 思路

**本题的难点在于如何定义`dp`数组的意义，以及如何进行状态转移**。

先分析这个原问题，想办法将其转换成规模较小的子问题。我们以 `A = horse`， `B = ros`为例子，看看如何将其转换为规模较小的子问题。

根据题意，我们可以对任意一个单词进行三种操作

- 插入一个字符
- 删除一个字符
- 替换一个字符

但我们可以发现，如果我们有单词 `A` 和单词 `B` ：

- **对单词 `A` 删除一个字符和对单词 `B` 插入一个字符是等价的**。例如当 `A = doge`，单词 `B = dog` 时，我们既可以删除 `A` 中的最后一个字符 `e` ，得到 `dog` ，也可以在单词 `B` 末尾添加一个字符 `e`，得到相同的 `doge`；
- 同理，**对单词 `B` 删除一个字符和对单词 `A` 插入一个字符也是等价的**。
- **对单词 `A` 替换一个字符和对单词 `B` 替换一个字符是等价的**。

这样一来，本质不同的操作实际上只有三种：

- 在 `A` 中插入一个字符
- 在 `B` 中插入一个字符
- 修改 `A` 的一个字符



于是，我们回到 `A = horse` ，`B = ros` 这个例子。

- **在 `A` 中插入一个字符**：如果我们知道 `B = horse` 到 `A = ro` 的编辑距离为 `a` ，那么显然 `horse` 到 `ros` 的编辑距离不会超过 `a+1` 。这是因为我们可以在 `a` 次操作后将 `horse` 和 `ro` 变为相同的字符串后，再进行额外的 `1` 次操作：在 `A` 的末尾添加字符 `s` ，就能在 `a+1` 次操作后将 `horse` 和 `ro` 变为相同的字符串；
- **在 `B` 中插入一个字符**：如果我们知道 `B = hors` 到 `A = ros` 的编辑距离为 `b` ，那么显然 `horse` 到 `ros` 的编辑距离不会超过 `b+1` ，原因是可以先由`ros`经过`b`次操作得到`hors`，然后进行一次插入操作；
- **修改 `A` 的一个字符**：如果我们知道 `hors` 到 `ro` 的编辑距离为 `c`，那么显然 `horse` 到 `ros` 的编辑距离不会超过 `c+1` ，原因同上。

那么从 `horse` 变为 `ros` 的编辑距离应该为 `min(a+1, b+1, c+1)`



我们定义 `dp[i][j]` 表示`word1` 的前 `i` 个字母到 `word2` 的前 `j` 个字母所需要的编辑距离。

- 当 `word1[i] == word2[j]` 时，有 `dp[i][j] = dp[i-1][j-1]`
- 否则，`dp[i][j] = 1 + min(dp[i][j-1], dp[i-1][j], dp[i-1][j-1])`

```c++
#include<bits/stdc++.h>
using namespace std;

int solve(string word1, string word2) {
    int len1 = word1.length(), len2 = word2.length();
    vector<vector<int>> f(len1 + 1, vector<int>(len2 + 1));
    for (int i = 1; i <= len1; ++i) f[i][0] = i;
    for (int j = 1; j <= len2; ++j) f[0][j] = j;

    for (int i = 1; i <= len1; ++i) {
        for (int j = 1; j <= len2; ++j) {
            if (word1[i-1] == word2[j-1]) {
                f[i][j] = f[i-1][j-1];
            } else {
                // 枚举插入、删除、替换三种情况
                f[i][j] = 1 + min({f[i-1][j], f[i][j-1], f[i-1][j-1]});
            }
        }
    }
    return f[len1][len2];
}

int main() {
    string s1 = "horse", s2 = "ros";
    cout << solve(s1, s2) << endl;
    string s3 = "intention", s4 = "execution";
    cout << solve(s3, s4) << endl;
    return 0;
}
```





## 74.岛屿数量（DFS）

给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围。

 

**示例 1：**

```
输入：grid = [
  ["1","1","1","1","0"],
  ["1","1","0","1","0"],
  ["1","1","0","0","0"],
  ["0","0","0","0","0"]
]
输出：1
```

**示例 2：**

```
输入：grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
输出：3
```

 

**提示：**

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 300`
- `grid[i][j]` 的值为 `'0'` 或 `'1'`



### 思路

参考这篇blog：[岛屿类问题的通用解法、DFS遍历框架 (yuk1pedia.github.io)](https://yuk1pedia.github.io/2024/07/DFS_traversal_framework/)

```c++
class Solution {
public:
    vector<pair<int, int>> dir = { {-1, 0}, {1, 0}, {0, -1}, {0, 1} };
    void dfs(vector<vector<char>>& grid, int x, int y) {
        if (!judge(grid, x, y)) return;
        if (grid[x][y] == '1') {
            grid[x][y] = '2';
            for (int i = 0; i < 4; ++i) {
                int dirx = x + dir[i].first;
                int diry = y + dir[i].second;
                dfs(grid, dirx, diry);
            }
        }
    }

    bool judge(vector<vector<char>>& grid, int x, int y) {
        int row = grid.size();
        int col = grid[0].size();
        if (x >= row || x < 0 || y >= col || y < 0) return false;
        return true;
    }

    int numIslands(vector<vector<char>>& grid) {
        int ans = 0;
        int row = grid.size(), col = grid[0].size();
        for (int i = 0; i < row; ++i) {
            for (int j = 0; j < col; ++j) {
                if (grid[i][j] == '1') {
                    dfs(grid, i, j);
                    ans++;
                }
            }
        }
        return ans;
    }
};
```





## 75.腐烂的橘子（BFS）

在给定的 `m x n` 网格 `grid` 中，每个单元格可以有以下三个值之一：

- 值 `0` 代表空单元格；
- 值 `1` 代表新鲜橘子；
- 值 `2` 代表腐烂的橘子。

每分钟，腐烂的橘子 **周围 4 个方向上相邻** 的新鲜橘子都会腐烂。

返回 *直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`* 。

 

**示例 1：**

![image.png](https://s2.loli.net/2025/02/18/RUQ1yawrgTiF8PD.png)

```
输入：grid = [[2,1,1],[1,1,0],[0,1,1]]
输出：4
```

**示例 2：**

```
输入：grid = [[2,1,1],[0,1,1],[1,0,1]]
输出：-1
解释：左下角的橘子（第 2 行， 第 0 列）永远不会腐烂，因为腐烂只会发生在 4 个方向上。
```

**示例 3：**

```
输入：grid = [[0,2]]
输出：0
解释：因为 0 分钟时已经没有新鲜橘子了，所以答案就是 0 。
```

 

**提示：**

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 10`
- `grid[i][j]` 仅为 `0`、`1` 或 `2`



### 思路

BFS可以看成是层序遍历。从某个节点出发，BFS首先遍历到距离为1的节点，然后是距离为2、3、4...的节点。因此，BFS可以用来求**最短路径问题**。BFS搜索到的节点，一定是距离最近的节点。

再来看这道题的要求：返回直到单元格中没有新鲜橘子为止所必需经过的最小分钟数。翻译一下，实际上就是求**腐烂橘子到所有新鲜橘子的最短路径**。那么这道题就可以使用BFS来解决。



**如何写（最短路径的）BFS代码？**

- BFS需要使用到队列这种数据结构，代码框架是这样的：

```python
while queue 非空:
	node = queue.pop()
    for node 的所有相邻结点 m:
        if m 未访问过:
            queue.push(m)
```

- 但是用BFS来求最短路径的话，这个队列中第 1 层和第 2 层的节点会紧挨在一起，无法区分当前节点是第 1 层还是第 2 层。因此，我们需要在每一层遍历开始前，记录队列中的节点数量 `n` ，然后一口气处理完这一层的 `n` 个节点。

```python
depth = 0 # 记录遍历到第几层
while queue 非空:
    depth++
    n = queue 中的元素个数
    循环 n 次:
        node = queue.pop()
        for node 的所有相邻结点 m:
            if m 未访问过:
                queue.push(m)
```



**本题主要思路**

- 一开始，我们找出所有腐烂的橘子，将它们放入队列，作为第 0 层的节点
- 然后进行BFS，每个节点的相邻节点可能是上、下、左、右四个方向的节点，注意判断越界情况
- 由于可能存在无法被污染的橘子（无法被BFS搜索到），我们需要记录新鲜橘子数量 `fresh` 。在BFS中，每遍历到一个新鲜橘子，就将新鲜橘子数量减一。如果BFS结束后这个数量仍未减为0，说明存在无法被污染的橘子

```c++
class Solution {
public:
    vector<pair<int, int>> dir = { {-1, 0}, {1, 0}, {0, -1}, {0, 1} };
    int orangesRotting(vector<vector<int>>& grid) {
        int ans = 0, fresh = 0; // fresh记录新鲜橘子数
        queue<pair<int, int>> q;
        int row = grid.size(), col = grid[0].size();
        for (int i = 0; i < row; ++i) {
            for (int j = 0; j < col; ++j) {
                if (grid[i][j] == 1) fresh++;
                else if (grid[i][j] == 2) q.push({i, j});
            }
        }

        // 开始bfs
        while (!q.empty()) {
            int n = q.size();
            bool rotten = false;
            for (int i = 0; i < n; ++i) {
                auto x = q.front();
                q.pop();
                for (auto cur : dir) {
                    int dirx = x.first + cur.first;
                    int diry = x.second + cur.second;
                    if (dirx >= 0 && dirx < row && diry >= 0 && diry < col
                    && grid[dirx][diry] == 1) {
                        grid[dirx][diry] = 2; // 新鲜橘子被腐烂
                        q.push({dirx, diry});
                        fresh--;
                        rotten = true;
                    }
                }
            }
            if (rotten) ans++;
        }
        return fresh ? -1 : ans;
    }
};
```





## 76.课程表

你这个学期必须选修 `numCourses` 门课程，记为 `0` 到 `numCourses - 1` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `prerequisites` 给出，其中 `prerequisites[i] = [ai, bi]` ，表示如果要学习课程 `ai` 则 **必须** 先学习课程 `bi` 。

- 例如，先修课程对 `[0, 1]` 表示：想要学习课程 `0` ，你需要先完成课程 `1` 。

请你判断是否可能完成所有课程的学习？如果可以，返回 `true` ；否则，返回 `false` 。

 

**示例 1：**

```
输入：numCourses = 2, prerequisites = [[1,0]]
输出：true
解释：总共有 2 门课程。学习课程 1 之前，你需要完成课程 0 。这是可能的。
```

**示例 2：**

```
输入：numCourses = 2, prerequisites = [[1,0],[0,1]]
输出：false
解释：总共有 2 门课程。学习课程 1 之前，你需要先完成课程 0 ；并且学习课程 0 之前，你还应先完成课程 1 。这是不可能的。
```

 

**提示：**

- `1 <= numCourses <= 2000`
- `0 <= prerequisites.length <= 5000`
- `prerequisites[i].length == 2`
- `0 <= ai, bi < numCourses`
- `prerequisites[i]` 中的所有课程对 **互不相同**





### 思路

本题是一道经典的**拓扑排序**问题。

给定一个包含 `n` 个节点的有向图 `G` ，我们给出它的节点编号的一种排列，如果满足：**对于图 `G` 中的任意一条有向边 `(u, v)` ，`u` 在排列中都出现在 `v` 的前面**，那么称该排列是图 `G` 的**拓扑排序**。

根据上述定义，我们可以得出两个结论：

- 如果图 `G` 中存在环（即图 `G` 不是**有向无环图**），那么图 `G` 不存在拓扑排序。这是因为假设图中存在环 `x1、x2、...、xn、x1` ，那么 `x1` 在排列中必须出现在 `xn` 前面，但 `xn` 同时也必须出现在 `x1` 的前面，因此不存在一个满足要求的排列，也就不存在拓扑排序；
- 如果图 `G` 是**有向无环图**，那么它的拓扑排序可能不止一种。举一个最极端的例子，如果图 `G` 值包含 `n` 个节点却**没有任何边**，**那么任意一种编号的排列都可以作为拓扑排序**。

有了上述的分析，我们就可以将本题建模成一个求拓扑排序的问题了：

- 我们将每一门课看成一个节点；
- 如果想要学习课程 `A` 之前必须完成课程 `B` ，那么我们从 `B` 到 `A` 连接一条有向边。这样以来，在拓扑排序中，`B` 一定出现在 `A` 的前面。

求出该图是否存在拓扑排序，就可以判断是否有一种符合要求的课程学习顺序。



**`DFS` 做法：**

用一个栈来存储已经**搜索完成的节点**，对于一个节点 `u` ，规定它的**相邻节点指的是从 `u` 出发通过一条有向边可以到达的所有节点**。如果它的所有相邻节点都已经搜索完成，那么在搜索回溯到 `u` 时，`u` 本身也会变成一个已经搜索完成的节点。

假设我们当前搜索到了节点 `u` ，如果它的所有相邻节点都已经搜索完成，那么这些节点就都已经在栈中了，此时我们就可以把 `u` 入栈。此时，我们从栈顶往栈底的顺序看，由于 `u` 处于栈顶的位置，那么 `u` 出现在所有 `u` 的相邻节点的前面。因此对于 `u` 而言，它是满足拓扑排序的要求的。

这样一来，我们对图进行一次 `DFS` ，当每个节点进行回溯的时候，我们把该节点放入栈中。最终从栈顶到栈底的序列就是一种拓扑排序。

对于图中的任意一个节点，它在搜索的过程中有三种状态：

- **未搜索**：还没有搜索过这个节点
- **搜索中**：搜索过这个节点，但还没有回溯到该节点（该节点还没有入栈）
- **已完成**：搜索过并且回溯过这个节点（该节点已经入栈）

于是，具体的算法为：在每一轮的搜索开始时，我们任取一个**未搜索**的节点开始 `DFS` 。

- 我们将当前搜索的节点 `u` 标记为**搜索中**，遍历该节点的每一个相邻节点 `v` ：
  - 如果 `v`  **未搜索**，那么开始搜索 `v` ，待搜索完成回溯到 `u` ；
  - 如果 `v`  **搜索中**，那么就找到了图中的一个环，此时拓扑排序不存在；
  - 如果 `v`  **已完成**，那么 `v` 已经在栈中了，`u` 无论何时入栈都不影响 `(u, v)` 之间的拓扑关系，不用进行任何操作
- 当 `u` 的所有相邻节点都为**已完成**时，我们将 `u` 放入栈中，并将其标记为**已完成**。

在整个 `DFS` 结束后，如果没有找到图中的环，那么栈中存储这所有的 `n` 个节点，从栈顶到栈底的顺序即为一种拓扑排序。

```c++
class Solution {
private:
    vector<vector<int>> edges; // 模拟栈
    vector<int> visited; // 0 表示未搜索；1 表示搜索中；2 表示已搜索
    bool valid = true;

public:
    void dfs(int u) {
        visited[u] = 1; // 搜索中
        for (auto v: edges[u]) {
            if (visited[v] == 0) {
                dfs(v);
                if (!valid) return; // 图中存在环
            } else if (visited[v] == 1) { // 图中存在环
                valid = false;
                return;
            }
        }
        visited[u] = 2; // 已搜索
    }
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        edges.resize(numCourses);
        visited.resize(numCourses);
        // 相邻节点入栈
        for (const auto& info: prerequisites) {
            edges[info[1]].push_back(info[0]); 
        }
        // 搜索整个图
        for (int i = 0; i < numCourses && valid; ++i) {
            if (!visited[i]) dfs(i);
        }
        return valid;
    }
};
```





**`BFS` 做法：**

`DFS` 是逆向思维：最先被放入栈中的节点是在拓扑排序中最后面的节点，也可以用正向思维，顺序的生成拓扑排序，这种方法也更加直观。

考虑拓扑排序中最前面的节点，**该节点的入度一定为 0** ，也就是它没有任何先修课程的要求。当我们将一个节点加入答案后，我们就可以移除它的所有出边，**代表它的相邻节点少了一门先修课程的要求**。如果某个相邻节点变成了**没有任何入边的节点**，就代表着这门课可以开始学习了。按照这样的流程，不断地将没有入边的节点加入答案，直到答案中包含所有的节点（得到了一种拓扑排序）或者不存在没有入边的节点（图中包含环）。

用一个队列来进行 `BFS` 。初始时，所有入度为 0 的节点被放入队列中，它们就是可以作为拓扑排序最前面的节点，并且它们之间的相对顺序是无关的。

在 `BFS` 的每一步中，取出队首节点 `u` ：

- 将 `u` 放入答案中；
- 移除 `u` 的所有出边，也就是将 `u` 的所有相邻节点的入度减少 1 。如果某个相邻节点 `v` 的入度变为 0 ，那么就把 `v` 放入队列中。

`BFS` 结束后，如果答案包含了 `n` 个节点，就找到了一种拓扑排序，否则说明图中有环。

```c++
class Solution {
private:
    vector<vector<int>> edges;
    vector<int> indeg; // 记录入度

public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        edges.resize(numCourses);
        indeg.resize(numCourses);
        for (const auto& info: prerequisites) {
            edges[info[1]].push_back(info[0]);
            ++indeg[info[0]]; // 记录节点入度
        }

        queue<int> q;
        // 入度为 0 的节点入队
        for (int i = 0; i < numCourses; ++i) {
            if (indeg[i] == 0) {
                q.push(i);
            }
        }

        int visited = 0; // 记录可以修的课程数
        while (!q.empty()) {
            ++visited;
            int u = q.front();
            q.pop();
            for (auto v: edges[u]) {
                --indeg[v];
                if (indeg[v] == 0) {
                    q.push(v);
                }
            }
        }
        return visited == numCourses;
    }
};
```





## 77.实现Trie(前缀树)

**[Trie](https://baike.baidu.com/item/字典树/9825209?fr=aladdin)**（发音类似 "try"）或者说 **前缀树** 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补完和拼写检查。

请你实现 Trie 类：

- `Trie()` 初始化前缀树对象。
- `void insert(String word)` 向前缀树中插入字符串 `word` 。
- `boolean search(String word)` 如果字符串 `word` 在前缀树中，返回 `true`（即，在检索之前已经插入）；否则，返回 `false` 。
- `boolean startsWith(String prefix)` 如果之前已经插入的字符串 `word` 的前缀之一为 `prefix` ，返回 `true` ；否则，返回 `false` 。

 

**示例：**

```
输入
["Trie", "insert", "search", "search", "startsWith", "insert", "search"]
[[], ["apple"], ["apple"], ["app"], ["app"], ["app"], ["app"]]
输出
[null, null, true, false, true, null, true]

解释
Trie trie = new Trie();
trie.insert("apple");
trie.search("apple");   // 返回 True
trie.search("app");     // 返回 False
trie.startsWith("app"); // 返回 True
trie.insert("app");
trie.search("app");     // 返回 True
```

 

**提示：**

- `1 <= word.length, prefix.length <= 2000`
- `word` 和 `prefix` 仅由小写英文字母组成
- `insert`、`search` 和 `startsWith` 调用次数 **总计** 不超过 `3 * 10^4` 次



```c++
class Trie {
private:
    vector<Trie*> children;
    bool isEnd;

    Trie* searchPrefix(string prefix) {
        Trie* node = this;
        for (auto ch: prefix) {
            ch -= 'a';
            if (node->children[ch] == nullptr) {
                return nullptr;
            }
            node = node->children[ch];
        }
        return node;
    }

public:
    Trie() : children(26), isEnd(false) {}
    
    void insert(string word) {
        Trie* node = this;
        for (auto ch: word) {
            ch -= 'a';
            if (node->children[ch] == nullptr) {
                node->children[ch] = new Trie();
            }
            node = node->children[ch];
        }
        node->isEnd = true;
    }
    
    bool search(string word) {
        Trie* node = this->searchPrefix(word);
        return node != nullptr && node->isEnd;
    }
    
    bool startsWith(string prefix) {
        return this->searchPrefix(prefix) != nullptr;
    }
};

/**
 * Your Trie object will be instantiated and called as such:
 * Trie* obj = new Trie();
 * obj->insert(word);
 * bool param_2 = obj->search(word);
 * bool param_3 = obj->startsWith(prefix);
 */
```





## 78.搜索插入位置

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



### 思路

基础二分搜索

```c++
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int n = nums.size();
        if (target > nums.back()) return n;
        int left = 0, right = n - 1;
        int mid = (left + right) / 2;
        while (left <= right) {
            if (target > nums[mid]) {
                left = mid + 1;
            } else if (target < nums[mid]) {
                right = mid - 1;
            } else {
                return mid;
            }
            mid = (left + right) / 2;
        }
        return left;
    }
};
```





## 79.搜索二维矩阵

给你一个满足下述两条属性的 `m x n` 整数矩阵：

- 每行中的整数从左到右按非严格递增顺序排列。
- 每行的第一个整数大于前一行的最后一个整数。

给你一个整数 `target` ，如果 `target` 在矩阵中，返回 `true` ；否则，返回 `false` 。

 

**示例 1：**

![image.png](https://s2.loli.net/2025/02/18/5Lhd9tbfcDKRuPr.png)

```
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
输出：true
```

**示例 2：**

![image.png](https://s2.loli.net/2025/02/18/X39Jz1LDBWM5cjH.png)

```
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13
输出：false
```

 

**提示：**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 100`
- `-10^4 <= matrix[i][j], target <= 10^4`



### 思路

先找目标值在哪一行，再对目标行进行二分搜索即可。代码如下：

```c++
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        auto binary = [&](vector<int>& nums, int target) -> int {
            int left = 0, right = nums.size() - 1;
            while (left <= right) {
                int mid = left + (right - left) / 2;
                if (nums[mid] < target) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
            return left;
        };

        int m = matrix.size(), n = matrix[0].size();
        for (int i = 0; i < m; ++i) {
            if (matrix[i][n-1] < target) {
                continue;
            }
            int index = binary(matrix[i], target);
            if (matrix[i][index] == target) return true;
        }
        return false;
    }
};
```





## 80.在排序数组中查找元素的第一个和最后一个位置

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题。

 

**示例 1：**

```
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]
```

**示例 2：**

```
输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]
```

**示例 3：**

```
输入：nums = [], target = 0
输出：[-1,-1]
```

 

**提示：**

- `0 <= nums.length <= 10^5`
- `-10^9 <= nums[i] <= 10^9`
- `nums` 是一个非递减数组
- `-10^9 <= target <= 10^9`



### 思路

题目要求时间复杂度为 `O(log n)` ，考虑二分搜索。下面是二分搜索的一种写法：

```c++
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        auto binary = [&](vector<int>& nums, int target) -> int {
            int left = 0, right = nums.size() - 1;
            while (left <= right) {
                int mid = left + (right - left) / 2;
                if (nums[mid] < target) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
            return left;
        };

        int start = binary(nums, target);
        if (start == nums.size() || nums[start] != target) {
            return {-1, -1}; // nums 中没有 target
        }
        // start 存在，end 必存在
        int end = binary(nums, target + 1) - 1;
        return {start, end};
    }
};
```

**注：在 `right = (int) nums.size() - 1` 处强转是防止溢出，`nums.size()` 返回的是无符号数，当该数很大时与有符号数运算可能会溢出；在 `mid = left + (right - left) / 2` 处也是为了防止溢出。**





## 81.搜索旋转排序数组

整数数组 `nums` 按升序排列，数组中的值 **互不相同** 。

在传递给函数之前，`nums` 在预先未知的某个下标 `k`（`0 <= k < nums.length`）上进行了 **旋转**，使数组变为 `[nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]`（下标 **从 0 开始** 计数）。例如， `[0,1,2,4,5,6,7]` 在下标 `3` 处经旋转后可能变为 `[4,5,6,7,0,1,2]` 。

给你 **旋转后** 的数组 `nums` 和一个整数 `target` ，如果 `nums` 中存在这个目标值 `target` ，则返回它的下标，否则返回 `-1` 。

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。

 

**示例 1：**

```
输入：nums = [4,5,6,7,0,1,2], target = 0
输出：4
```

**示例 2：**

```
输入：nums = [4,5,6,7,0,1,2], target = 3
输出：-1
```

**示例 3：**

```
输入：nums = [1], target = 0
输出：-1
```

 

**提示：**

- `1 <= nums.length <= 5000`
- `-10^4 <= nums[i] <= 10^4`
- `nums` 中的每个值都 **独一无二**
- 题目数据保证 `nums` 在预先未知的某个下标上进行了旋转
- `-10^4 <= target <= 10^4`



### 思路

二分搜索只能搜索有序数组，而题目中的数组在**某一部分是有序的**，所以我们可以在常规的二分搜索中，查看当前 `mid` 为分割位置分割出来的两部分：`[l, mid]` 和 `[mid+1, r]` 哪个部分是有序的，然后根据有序的那个部分确定如何改变二分的上下界：

- 如果 `[l, mid-1]` 有序，且 `target` 大小满足 `[nums[l], nums[mid])` ，则将搜索范围缩小到 `[l, mid-1]` ，否则在 `[mid+1, r]` 中寻找
- 如果 `[mid, r]` 有序，且 `target` 大小满足 `(nums[mid+1], nums[r]]` ，则将搜索范围缩小到 `[mid+1, r]` 否则在 `[l, mid-1]` 中寻找

代码如下

```c++
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int n = (int)nums.size();
        if (!n) return -1;
        if (n == 1) return nums[0] == target ? 0 : -1;
        int l = 0, r = n - 1;
        while (l <= r) {
            int mid = l + (r - l) / 2;
            if (nums[mid] == target) return mid;
            if (nums[0] <= nums[mid]) { // [l, mid-1] 有序
                if (nums[0] <= target && target < nums[mid]) { // 且 target 满足 [nums[l], nums[mid])
                    r = mid - 1;
                } else {
                    l = mid + 1;
                }
            } else { // [mid, r] 有序
                if (nums[mid] < target && target <= nums[n-1]) { // 且 target 满足 (nums[mid+1], nums[r]]
                    l = mid + 1;
                } else {
                    r = mid - 1;
                }
            }
        }
        return -1;
    }
};
```







## 82.寻找旋转排序数组中的最小值

已知一个长度为 `n` 的数组，预先按照升序排列，经由 `1` 到 `n` 次 **旋转** 后，得到输入数组。例如，原数组 `nums = [0,1,2,4,5,6,7]` 在变化后可能得到：

- 若旋转 `4` 次，则可以得到 `[4,5,6,7,0,1,2]`
- 若旋转 `7` 次，则可以得到 `[0,1,2,4,5,6,7]`

注意，数组 `[a[0], a[1], a[2], ..., a[n-1]]` **旋转一次** 的结果为数组 `[a[n-1], a[0], a[1], a[2], ..., a[n-2]]` 。

给你一个元素值 **互不相同** 的数组 `nums` ，它原来是一个升序排列的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的 **最小元素** 。

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。

 

**示例 1：**

```
输入：nums = [3,4,5,1,2]
输出：1
解释：原数组为 [1,2,3,4,5] ，旋转 3 次得到输入数组。
```

**示例 2：**

```
输入：nums = [4,5,6,7,0,1,2]
输出：0
解释：原数组为 [0,1,2,4,5,6,7] ，旋转 3 次得到输入数组。
```

**示例 3：**

```
输入：nums = [11,13,15,17]
输出：11
解释：原数组为 [11,13,15,17] ，旋转 4 次得到输入数组。
```

 

**提示：**

- `n == nums.length`
- `1 <= n <= 5000`
- `-5000 <= nums[i] <= 5000`
- `nums` 中的所有整数 **互不相同**
- `nums` 原来是一个升序排序的数组，并进行了 `1` 至 `n` 次旋转



### 思路

时间复杂度需要 `O(log n)` ，说明需要使用二分查找。

设 `x = nums[mid]` 是我们当前二分取到的数，我们需要判断 `x` 和**数组最小值**的关系，谁在左边、谁在右边？

我们把 `x` 和最后一个数 `nums[n-1]` 比大小，可以得到下面的结论：

- 如果 `x > nums[n-1]` ，有：
  - `nums` 一定被分为了左右两个递增段
  - 第一段的所有元素均大于第二段的所有元素
  - `x` 在第一段
  - 最小值在第二段
  - **所以 `x` 一定在最小值的左边**
- 如果 `x <= nums[n-1]` ，有：
  - `x` 要么是最小值，要么在最小值**右边**

所以我们只需要不断比较 `x` 和 `nums[n-1]` 的关系，就间接的知道 `x` 和数组最小值的位置关系，从而不断缩小数组的查找范围，二分查找到数组的最小值

```c++
class Solution {
public:
    int findMin(vector<int>& nums) {
        int left = 0, right = nums.size() - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] > nums.back()) { // 与最小值比较
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return nums[left];
    }
};
```





## 83.寻找两个正序数组的中位数

给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 **中位数** 。

算法的时间复杂度应该为 `O(log (m+n))` 。

 

**示例 1：**

```
输入：nums1 = [1,3], nums2 = [2]
输出：2.00000
解释：合并数组 = [1,2,3] ，中位数 2
```

**示例 2：**

```
输入：nums1 = [1,2], nums2 = [3,4]
输出：2.50000
解释：合并数组 = [1,2,3,4] ，中位数 (2 + 3) / 2 = 2.5
```

 

 

**提示：**

- `nums1.length == m`
- `nums2.length == n`
- `0 <= m <= 1000`
- `0 <= n <= 1000`
- `1 <= m + n <= 2000`
- `-10^6 <= nums1[i], nums2[i] <= 10^6`



### 思路

同样需要使用二分的方法。我们要寻找的是中位数，以下面的数据为例：

![image.png](https://s2.loli.net/2025/02/18/dHXt73oTK4h6Flk.png)

寻找这两个数组的中位数，就是寻找两个有序数组中第 `k` 小的数，其中 `k` 是中位数的下标，在上面的例子中 `k = 7` 。为了找到第 `k` 小的数，我们可以分别找两个数组中第 `k/2` 个数，如果 `k` 是奇数就向下取整。

比较的过程中，**哪个小就表明该数组前 `k/2` 个数字都不是第 `k` 小的数字**，所以可以排除。在上述例子中，我们就要把第二个数组中的 `[1, 2, 3]` 排除掉，将 `[1, 3, 4, 9]` 和 `[4, 5, 6, 7, 8, 9, 10]` 两个数组作为新的数组进行比较。

由于我们已经排除掉了 3 个数字，那么这 3 个数字一定是在最前面的，所以在两个新数组中我们只需要找到第 `7 - 3 = 4` 小的数字就可以了，也就是 `k = 4` 。同样我们分别找到两个数组中第 `k/2` 个数，此时两个数组如下（橙色表示已经排除掉的数字）：

![image.png](https://s2.loli.net/2025/02/18/k3mASIrWbNpun9O.png)

此时就可以排除第一个数组的 `[1, 3]` ，于是乎 `k` 再减小 2 ，此时 `k = 2` ，得到如下情况：

![image.png](https://s2.loli.net/2025/02/18/sHy98nlgXLWchRx.png)

此时发现两个数组第 `k/2` 小的数相等，此时我们随便去掉一个都行，这里去掉第二个数组的 4 ，得到下面的情况：

![image.png](https://s2.loli.net/2025/02/18/gmQ3vYwcCdMGKIF.png)

**此时 `k = 1` ，说明需要找到第 `1` 小的数，我们直接返回此时更小的数就是题目所需要的结果**

代码如下：

```c++
class Solution {
public:
    // 寻找两个正序数组中第 k 小的数字
    int getKthElement(const vector<int>& nums1, const vector<int>& nums2, int k) {
        int m = nums1.size();
        int n = nums2.size();
        int index1 = 0, index2 = 0;

        while (true) {
            if (index1 == m) { // nums1 中数字均排除
                return nums2[index2 + k - 1];
            }
            if (index2 == n) { // nums2 中数字均排除
                return nums1[index1 + k - 1];
            }
            if (k == 1) {
                return min(nums1[index1], nums2[index2]);
            }

            // 正常情况
            int newIndex1 = min(index1 + k / 2 - 1, m - 1);
            int newIndex2 = min(index2 + k / 2 - 1, n - 1);
            int pivot1 = nums1[newIndex1];
            int pivot2 = nums2[newIndex2];
            if (pivot1 <= pivot2) {
                k -= newIndex1 - index1 + 1;
                index1 = newIndex1 + 1;
            }
            else {
                k -= newIndex2 - index2 + 1;
                index2 = newIndex2 + 1;
            }
        }
    }

    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int totalLength = nums1.size() + nums2.size();
        if (totalLength % 2 == 1) { // 奇数
            return getKthElement(nums1, nums2, (totalLength + 1) / 2);
        }
        else { // 偶数
            return (getKthElement(nums1, nums2, totalLength / 2) + getKthElement(nums1, nums2, totalLength / 2 + 1)) / 2.0;
        }
    }
};
```





## 84.买卖股票的最佳时机

给定一个数组 `prices` ，它的第 `i` 个元素 `prices[i]` 表示一支给定股票第 `i` 天的价格。

你只能选择 **某一天** 买入这只股票，并选择在 **未来的某一个不同的日子** 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 `0` 。

 

**示例 1：**

```
输入：[7,1,5,3,6,4]
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
```

**示例 2：**

```
输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 没有交易完成, 所以最大利润为 0。
```

 

**提示：**

- `1 <= prices.length <= 10^5`
- `0 <= prices[i] <= 10^4`



### 思路

两个思路，可以用 dp 也可以用贪心。

- dp 做法
  - 定义状态： `dp[i][0]` 表示第 `i` 天持有股票的最大利润，`dp[i][1]` 表示第 `i` 天不持有股票的最大利润（**注意此处持有表示拥有股票，并不一定指当天买入股票**）
  - 状态转移
    - 如果当天持有股票，就有两种情况：
      - 前一天就持有了股票，直接由前一天转移过来：`dp[i][0] = dp[i-1][0]`
      - 前一天并不持有股票，我们需要在当天购买股票：`dp[i][0] = -prices[i]` （**因为只能买卖一次股票，买入股票就一定是 `-prices[i]` 的利润**）
      - 在以上两种情况取最大值即可：` dp[i][0] = max(dp[i-1][0], -prices[i]);`
    - 如果当天不持有股票，也有两种情况：
      - 前一天就不持有股票：`dp[i][1] = dp[i-1][1]`
      - 前一天持有股票，这一天卖出：`dp[i][1] = dp[i-1][0] + prices[i]`
      - 两种情况取最大：`dp[i][1] = max(dp[i-1][1], dp[i-1][0]+prices[i]);`
  - 边界定义
    - 初始化第 0 天即可

代码如下：

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        int dp[n][2]; // dp[i][0]表示第 i 天持有的最大利润，dp[i][1]表示第 i 天不持有的最大利润
        dp[0][0] = -prices[0], dp[0][1] = 0;
        for (int i = 1; i < n; ++i) {
            dp[i][0] = max(dp[i-1][0], -prices[i]);
            dp[i][1] = max(dp[i-1][1], dp[i-1][0]+prices[i]);
        }
        return max(dp[n-1][0], dp[n-1][1]);
    }
};
```



- 贪心做法
  - 遍历一次，实时更新最小值与答案

代码如下：

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        int minn =INT_MAX, ans = 0;
        for (int i = 0; i < n; ++i) {
            minn = min(minn, prices[i]);
            ans = max(prices[i] - minn, ans);
        }
        return ans;
    }
};
```





## 85.最小栈

设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

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
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.getMin();   --> 返回 -2.
```

 

**提示：**

- `-2^31 <= val <= 2^31 - 1`
- `pop`、`top` 和 `getMin` 操作总是在 **非空栈** 上调用
- `push`, `pop`, `top`, and `getMin`最多被调用 `3 * 10^4` 次



### 思路

基础操作都可以通过 c++ 自带的 stack 实现，为了实现常数时间内检索到最小元素，只需要另外开一个栈即可。在这个新的栈中放最小的元素，调用返回时直接返回最小栈的栈顶元素，为了保证栈非空，最小栈需要与已有的栈同步压入弹出元素。代码如下

```c++
class MinStack {
private:
    stack<int> x_stack;
    stack<int> min_stack;
public:
    MinStack() {}
    
    void push(int val) {
        x_stack.push(val);
        if (min_stack.empty()) min_stack.push(val);
        else min_stack.push(min(min_stack.top(), val));
    }
    
    void pop() {
        x_stack.pop();
        min_stack.pop();
    }
    
    int top() {
        return x_stack.top();
    }
    
    int getMin() {
        return min_stack.top();
    }
};

/**
 * Your MinStack object will be instantiated and called as such:
 * MinStack* obj = new MinStack();
 * obj->push(val);
 * obj->pop();
 * int param_3 = obj->top();
 * int param_4 = obj->getMin();
 */
```





## 86.字符串解码

给定一个经过编码的字符串，返回它解码后的字符串。

编码规则为: `k[encoded_string]`，表示其中方括号内部的 `encoded_string` 正好重复 `k` 次。注意 `k` 保证为正整数。

你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。

此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 `k` ，例如不会出现像 `3a` 或 `2[4]` 的输入。

 

**示例 1：**

```
输入：s = "3[a]2[bc]"
输出："aaabcbc"
```

**示例 2：**

```
输入：s = "3[a2[c]]"
输出："accaccacc"
```

**示例 3：**

```
输入：s = "2[abc]3[cd]ef"
输出："abcabccdcdcdef"
```

**示例 4：**

```
输入：s = "abc3[cd]xyz"
输出："abccdcdcdxyz"
```

 

**提示：**

- `1 <= s.length <= 30`
- `s` 由小写英文字母、数字和方括号 `'[]'` 组成
- `s` 保证是一个 **有效** 的输入。
- `s` 中所有整数的取值范围为 `[1, 300]` 





### 思路

括号匹配类问题往往都会尝试去用栈解决，本题需要根据左右括号的匹配来进行字符串的展开。由于展开还需要看重复次数，我们用**两个栈**分别记录**重复次数**以及**需要重复的字符串**。用两个栈可以保证内部括号的字符串与重复次数相对应。

先遍历一遍字符串

- 如果遇到数字，就将字符串转换为数字 `num` 并压入数字栈中
- 如果遇到左括号 `'['` ，把 `num` 和 `res` 分别压入数字栈和字符串栈中
- 如果遇到字符，就把该字符拼接到已有的字符串 `res` 后面
- 如果遇到右括号 `']'` ，根据数字栈栈顶元素重复字符串栈栈顶元素，重复完后弹栈

代码如下：

```c++
class Solution {
public:
    string decodeString(string s) {
        string res = "";
        stack<int> nums; // 记录重复次数
        stack<string> strs; // 记录需要重复的字符串
        int num = 0, len = s.length();
        for (int i = 0; i < len; ++i) {
            if (isdigit(s[i])) {
                num = num * 10 + s[i] - '0';
            } else if (isalpha(s[i])) {
                res += s[i];
            } else if (s[i] == '[') { // 将左括号前的数字和字符串分别入栈
                nums.push(num);
                num = 0;
                strs.push(res);
                res = "";
            } else { // 遇见右括号，进行拼接
                int times = nums.top();
                nums.pop();
                for (int j = 0; j < times; ++j) {
                    // 此时 res 中是需要重复的字符串，strs.top() 是已经重复完的字符串
                    strs.top() += res;
                }
                res = strs.top();
                strs.pop();
            }
        }
        return res;
    }
};
```





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



### 补充

在**单调栈基础题**中，经常需要类似这种的解题思路：**在 `O(n)` 的时间复杂度内求出数组中各个元素右侧第一个更大的元素及其下标，然后一并得到其他信息**，均可以往本题的做法上去思考。





## 88.有效的括号

给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。
3. 每个右括号都有一个对应的相同类型的左括号。

 

**示例 1：**

```
输入：s = "()"
输出：true
```

**示例 2：**

```
输入：s = "()[]{}"
输出：true
```

**示例 3：**

```
输入：s = "(]"
输出：false
```

 

**提示：**

- `1 <= s.length <= 10^4`
- `s` 仅由括号 `'()[]{}'` 组成



### 思路

栈的基本应用，代码如下

```c++
class Solution {
public:
    bool isValid(string s) {
        if (s.length() == 1) return false;
        stack<char> st; 
        for (auto c: s) {
            if (c == '(' || c == '{' || c == '[') {
                st.push(c);
            } else {
                if (!st.empty()) {
                    if (c == ')') {
                        if (st.top() != '(') return false;
                        st.pop();
                    } else if (c == '}') {
                        if (st.top() != '{') return false;
                        st.pop();
                    } else if (c == ']') {
                        if (st.top() != '[') return false;
                        st.pop();
                    }
                } else {
                    return false;
                }
            }
        }
        if (st.empty()) return true;
        return false;
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





## 90.数组中的第K个最大元素

给定整数数组 `nums` 和整数 `k`，请返回数组中第 `**k**` 个最大的元素。

请注意，你需要找的是数组排序后的第 `k` 个最大的元素，而不是第 `k` 个不同的元素。

你必须设计并实现时间复杂度为 `O(n)` 的算法解决此问题。

 

**示例 1:**

```
输入: [3,2,1,5,6,4], k = 2
输出: 5
```

**示例 2:**

```
输入: [3,2,3,1,2,4,5,5,6], k = 4
输出: 4
```

 

**提示：**

- `1 <= k <= nums.length <= 10^5`
- `-10^4 <= nums[i] <= 10^4`



### 思路

堆排序，时间复杂度 `O(logn)` 。

```c++
class Solution {
public:
    void maxHeapify(vector<int>& a, int i, int heapSize) {
        int l = i * 2 + 1, r = i * 2 + 2, largest = i;
        if (l < heapSize && a[l] > a[largest]) {
            largest = l;
        } 
        if (r < heapSize && a[r] > a[largest]) {
            largest = r;
        }
        if (largest != i) {
            swap(a[i], a[largest]);
            maxHeapify(a, largest, heapSize);
        }
    }

    void buildMaxHeap(vector<int>& a, int heapSize) {
        for (int i = heapSize / 2; i >= 0; --i) {
            maxHeapify(a, i, heapSize);
        } 
    }

    int findKthLargest(vector<int>& nums, int k) {
        int heapSize = nums.size();
        buildMaxHeap(nums, heapSize);
        for (int i = nums.size() - 1; i >= nums.size() - k + 1; --i) {
            swap(nums[0], nums[i]);
            --heapSize;
            maxHeapify(nums, 0, heapSize);
        }
        return nums[0];
    }
};
```

快速选择（类似快速排序），时间复杂度 `O(n)` 。

```c++
class Solution {
private:
    int quickSelect(vector<int>& nums, int k) {
        // 随机选择基准数
        int pivot = nums[rand() % nums.size()];
        // 将大于、小于、等于 pivot 的元素划分至 big, small, equal 中
        // 基准左边更大，基准右边更小
        vector<int> big, equal, small;
        for (int num : nums) {
            if (num > pivot)
                big.push_back(num);
            else if (num < pivot)
                small.push_back(num);
            else
                equal.push_back(num);
        }

        // 注意此处是基准数左侧的数更小，右侧的数更大
        // 第 k 大元素在 big 中，递归划分
        if (k <= big.size()) {
            // 在 big 中找第 k 大元素
            return quickSelect(big, k);
        }
        // 第 k 大元素在 small 中，递归划分
        if (nums.size() - small.size() < k) {
            //在 small 中找第 k 大
            return quickSelect(small, k - (nums.size() - small.size()));
        }

        // 第 k 大元素在 equal 中，直接返回 pivot
        return pivot;
    }

public:
    int findKthLargest(vector<int>& nums, int k) {
        return quickSelect(nums, k);
    }
};
```





## 91.前 K 个高频元素

给你一个整数数组 `nums` 和一个整数 `k` ，请你返回其中出现频率前 `k` 高的元素。你可以按 **任意顺序** 返回答案。

 

**示例 1:**

```
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

**示例 2:**

```
输入: nums = [1], k = 1
输出: [1]
```

 

**提示：**

- `1 <= nums.length <= 105`
- `k` 的取值范围是 `[1, 数组中不相同的元素的个数]`
- 题目数据保证答案唯一，换句话说，数组中前 `k` 个高频元素的集合是唯一的

 

**进阶：**你所设计算法的时间复杂度 **必须** 优于 `O(n log n)` ，其中 `n` 是数组大小。



### 思路

可以遍历数组并记录每个数出现的次数，再进行排序判断。但这个复杂度是 `O(nlogn)` ，不满足题目要求。

这里可以用堆的思想：建立一个小顶堆，然后遍历**出现次数数组**

- 如果堆的元素个数小于 `k` ，就直接插入到堆中；
- 如果堆的元素个数等于 `k` ，则检查堆顶于当前出现次数的大小。如果堆顶更大，说明至少有 `k` 个数字出现次数比当前值大，故舍弃当前值；否则就弹出堆顶，并将当前值插入堆中

遍历完成后，堆中元素就代表了**出现次数数组**中前 `k` 大的值

```c++
class Solution {
public:
    static bool cmp(pair<int, int>& m, pair<int, int>& n) {
        return m.second > n.second;
    }

    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> occurrences;
        for (auto& v: nums) {
            occurrences[v]++;
        }

        priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(&cmp)> q(cmp);
        for (auto& [num, count] : occurrences) {
            if (q.size() == k) {
                if (q.top().second < count) {
                    q.pop();
                    q.emplace(num, count);
                }
            } else {
                q.emplace(num, count);
            }
        }
        vector<int> ret;
        while (!q.empty()) {
            ret.emplace_back(q.top().first);
            q.pop();
        }
        return ret;
    }
};
```







## 92.数据流的中位数

**中位数**是有序整数列表中的中间值。如果列表的大小是偶数，则没有中间值，中位数是两个中间值的平均值。

- 例如 `arr = [2,3,4]` 的中位数是 `3` 。
- 例如 `arr = [2,3]` 的中位数是 `(2 + 3) / 2 = 2.5` 。

实现 MedianFinder 类:

- `MedianFinder() `初始化 `MedianFinder` 对象。
- `void addNum(int num)` 将数据流中的整数 `num` 添加到数据结构中。
- `double findMedian()` 返回到目前为止所有元素的中位数。与实际答案相差 `10-5` 以内的答案将被接受。

**示例 1：**

```
输入
["MedianFinder", "addNum", "addNum", "findMedian", "addNum", "findMedian"]
[[], [1], [2], [], [3], []]
输出
[null, null, null, 1.5, null, 2.0]

解释
MedianFinder medianFinder = new MedianFinder();
medianFinder.addNum(1);    // arr = [1]
medianFinder.addNum(2);    // arr = [1, 2]
medianFinder.findMedian(); // 返回 1.5 ((1 + 2) / 2)
medianFinder.addNum(3);    // arr[1, 2, 3]
medianFinder.findMedian(); // return 2.0
```

**提示:**

- `-10^5 <= num <= 10^5`
- 在调用 `findMedian` 之前，数据结构中至少有一个元素
- 最多 `5 * 10^4` 次调用 `addNum` 和 `findMedian`



### 思路

比如现在有 6 个数：`1,5,6,2,3,4`，要计算中位数，可以把这 6 个数从小到大排序，得到 `1,2,3,4,5,6`，中间 3 和 4 的平均值 3.5 就是中位数。

中位数把这 6 个数均分成了左右两部分，一边是 `left=[1,2,3]`，另一边是 `right=[4,5,6]`。我们要计算的中位数，就来自 *left* 中的最大值，以及 *right* 中的最小值。

随着 *addNum* 不断地添加数字，我们需要：

- 保证 *left* 的大小和 *right* 的大小尽量相等。**规定**：在有奇数个数时，*left* 比 *right* 多 1 个数。
- 保证 *left* 的所有元素都小于等于 *right* 的所有元素。

只要时时刻刻满足以上两个要求，我们就可以用 *left* 的最大值以及 *right* 的最小值计算中位数。

分类讨论：

- 如果当前 *left* 的大小和 *right* 的大小相等：
  - 如果添加的数字 *num* 比较大，比如添加 7，那么把 7 加到 *right* 中。现在 *left* 比 *right* 少 1 个数，不符合前文的规定，所以必须把 *right* 的最小值从 *right* 中去掉，添加到 *left* 中。如此操作后，可以保证 *left* 的所有元素都小于等于 *right* 的所有元素。
  - 如果添加的数字 *num* 比较小，比如添加 0，那么把 0 加到 *left* 中。
  - 这两种情况可以**合并**：无论 *num* 是大是小，都可以先把 *num* 加到 *right* 中，然后把 *right* 的最小值从 *right* 中去掉，并添加到 *left* 中。
- 如果当前 *left* 比 *right* 多 1 个数：
  - 如果添加的数字 *num* 比较大，比如添加 7，那么把 7 加到 *right* 中。
  - 如果添加的数字 *num* 比较小，比如添加 0，那么把 0 加到 *left* 中。现在 *left* 比 *right* 多 2 个数，不符合前文的规定，所以必须把 *left* 的最大值从 *left* 中去掉，添加到 *right* 中。如此操作后，可以保证 *left* 的所有元素都小于等于 *right* 的所有元素。
  - 这两种情况可以**合并**：无论 *num* 是大是小，都可以先把 *num* 加到 *left* 中，然后把 *left* 的最大值从 *left* 中去掉，并添加到 *right* 中。

最后，我们需要什么样的数据结构？这个数据结构需要能高效地执行如下操作：

- 添加元素
- 找到最大（小）值
- 删除最大（小）值

这个数据结构是**堆**。

*left* 是**最大堆**，*right* 是**最小堆**。

- 如果当前有奇数个元素，中位数是 *left* 的堆顶
- 如果当前有偶数个元素，中位数是 *left* 的堆顶和 *right* 的堆顶的平均值

```c++
class MedianFinder {
private:
    priority_queue<int> left; // 最大堆
    priority_queue<int, vector<int>, greater<int>> right; //最小堆

public:
    MedianFinder() {}
    
    void addNum(int num) {
        if (left.size() == right.size()) {
            right.push(num);
            left.push(right.top());
            right.pop();
        } else {
            left.push(num);
            right.push(left.top());
            left.pop();
        }
    }
    
    double findMedian() {
        if (left.size() > right.size()) {
            return left.top();
        }
        return (left.top() + right.top()) / 2.0;
    }
};

/**
 * Your MedianFinder object will be instantiated and called as such:
 * MedianFinder* obj = new MedianFinder();
 * obj->addNum(num);
 * double param_2 = obj->findMedian();
 */
```





## 93.跳跃游戏

给你一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标，如果可以，返回 `true` ；否则，返回 `false` 。

 

**示例 1：**

```
输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。
```

**示例 2：**

```
输入：nums = [3,2,1,0,4]
输出：false
解释：无论怎样，总会到达下标为 3 的位置。但该下标的最大跳跃长度是 0 ， 所以永远不可能到达最后一个下标。
```

 

**提示：**

- `1 <= nums.length <= 10^4`
- `0 <= nums[i] <= 10^5`



### 思路

维护一个当前可以到达的最远距离 `max_dis` ，先初始化为 0 ，然后从头开始遍历数组。

**如果当前遍历到的下标在 `max_dis` 内**，那么更新可以达到的最远距离。

不断地重复该操作，直到 ：

- `max_dis >= nums.size() - 1` ，返回 `true` 。

- 如果数组遍历完 `max_dis` 仍然小于 `nums.size() - 1` ，就说明无法达到。

代码如下：

```c++
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int n = nums.size();
        int max_dis = 0;
        for (int i = 0; i < n; ++i) {
            if (i <= max_dis) {
                max_dis = max(max_dis, nums[i] + i);
                if (max_dis >= n - 1) return true;
            }
        }
        return false;
    }
};
```





## 94.跳跃游戏Ⅱ

给定一个长度为 `n` 的 **0 索引**整数数组 `nums`。初始位置为 `nums[0]`。

每个元素 `nums[i]` 表示从索引 `i` 向前跳转的最大长度。换句话说，如果你在 `nums[i]` 处，你可以跳转到任意 `nums[i + j]` 处:

- `0 <= j <= nums[i]` 
- `i + j < n`

返回到达 `nums[n - 1]` 的最小跳跃次数。生成的测试用例可以到达 `nums[n - 1]`。

 

**示例 1:**

```
输入: nums = [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
```

**示例 2:**

```
输入: nums = [2,3,0,1,4]
输出: 2
```

 

**提示:**

- `1 <= nums.length <= 10^4`
- `0 <= nums[i] <= 1000`
- 题目保证可以到达 `nums[n-1]`



### 思路

从头开始遍历数组，维护当前能到达的最远距离 `max_dis` ，**每次在上次能跳到的范围（end）内选择一个能跳的最远的位置（max_dis）作为下次的起跳点**。

![image.png](https://s2.loli.net/2025/02/18/FumEwoIQOXqWMVg.png) 

在遍历数组时，我们**不访问最后一个元素**，这是因为在访问最后一个元素之前，我们的边界一定大于等于最后一个位置，否则就无法跳到最后一个位置了。**如果访问最后一个元素，在边界正好为最后一个位置的情况下，我们会增加一次「不必要的跳跃次数」**，因此我们不必访问最后一个元素。

代码如下：

```c++
class Solution {
public:
    int jump(vector<int>& nums) {
        int n = nums.size();
        // max_dis 保存目前遍历到的能达到的最远位置
        // end 保存从上一个起跳点起跳能到达的右边界
        int res = 0, max_dis = 0, end = 0;
        for (int i = 0; i < n - 1; ++i) {
            // 如果当前遍历到的点在目前能达到的最远位置的范围内
            if (max_dis >= i) {
                // 更新能到达的最远位置
                max_dis = max(max_dis, i + nums[i]);
                // 如果当前到达了上次起跳能到达的右边界，更新右边界 end 为当前能达到的最远位置
                if (i == end) {
                    // 更新 end 相当于从上次能跳到的范围（end）内选择一个能跳的最远的位置起跳
                    end = max_dis;
                    ++res; // 进行下一次起跳
                }
            }
        }
        return res;
    }
};
```







## 95.划分字母区间

给你一个字符串 `s` 。我们要把这个字符串划分为尽可能多的片段，同一字母最多出现在一个片段中。

注意，划分结果需要满足：将所有划分结果按顺序连接，得到的字符串仍然是 `s` 。

返回一个表示每个字符串片段的长度的列表。

 

**示例 1：**

```
输入：s = "ababcbacadefegdehijhklij"
输出：[9,7,8]
解释：
划分结果为 "ababcbaca"、"defegde"、"hijhklij" 。
每个字母最多出现在一个片段中。
像 "ababcbacadefegde", "hijhklij" 这样的划分是错误的，因为划分的片段数较少。 
```

**示例 2：**

```
输入：s = "eccbbbbdec"
输出：[10]
```

 

**提示：**

- `1 <= s.length <= 500`
- `s` 仅由小写英文字母组成



### 思路

每个字母在每个片段中仅仅出现一次，这需要我们记录每个字母在字符串 `s` 中最后出现的下标。我们先遍历一遍 `s` ，记录每个字母最后出现的下标 `endc = lase[s[i]-'a']`，然后再遍历一次 `s` ，定义 `start` 和 `end` 两个变量，用于表示当前字符串片段，对于每个字母 `c` ，`end` 一定不会比 `endc` 更小，于是不断地更新 `end` 为 `max(end, last[s[i]-'a'])` ，直到 `i == end` ，说明当前片段遍历结束，找到了一个最短的满足条件的字符串片段，将答案添加进数组中，直到整个字符串遍历完。

**上述做法用贪心的思想寻找每个片段可能的最小结束下标**，因此可以保证每个片段的长度一定是符合要求的最短长度

- 如果取更短的片段，则**一定**会出现**同一个字母出现在多个片段**的情况。由于每次取的片段都是符合要求的最短片段，因此得到的片段数也是最多的
- 由于每个片段访问结束的标志是访问到下标 `end` ，因此对于每个片段，**可以保证当前片段中的每个字母都一定只在当前片段中**，不可能出现在其他片段，可以保证同一个字母只会出现在同一个片段

代码如下：

```c++
class Solution {
public:
    vector<int> partitionLabels(string s) {
        vector<int> last(26);
        int len = s.length();
        // 记录每个字母最后出现的下标
        for (int i = 0; i < len; ++i) last[s[i]-'a'] = i;
        vector<int> res;
        int start = 0, end = 0;
        for (int i = 0; i < len; ++i) {
            end = max(end, last[s[i]-'a']);
            if (i == end) {
                res.push_back(end - start + 1);
                start = end + 1;
            }
        }
        return res;
    }
};
```







## 96.只出现一次的数字

给你一个 **非空** 整数数组 `nums` ，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

你必须设计并实现线性时间复杂度的算法来解决此问题，且该算法只使用常量额外空间。

 

**示例 1 ：**

```
输入：nums = [2,2,1]
输出：1
```

**示例 2 ：**

```
输入：nums = [4,1,2,1,2]
输出：4
```

**示例 3 ：**

```
输入：nums = [1]
输出：1
```

 

**提示：**

- `1 <= nums.length <= 3 * 10^4`
- `-3 * 10^4 <= nums[i] <= 3 * 10^4`
- 除了某个元素只出现一次以外，其余每个元素均出现两次。



### 思路

如果不限制常量的额外空间，可以用 `unordered_set` 进行筛选。

此题限制了常量的额外空间，由于要找到只出现一次的数字，这里可以考虑位运算：由于**相同的数做异或运算得 0** ，**任何数与 0 做异或运算得自身**。同时数组中只有一个出现一次的数字，我们可以把整个数组的数字和 0 做异或运算，最后得到的结果就是我们需要找的只出现一次的数字。

代码如下：

```c++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res = 0;
        for (const auto& num: nums) res ^= num;
        return res;
    }
};
```







## 97.多数元素

给定一个大小为 `n` 的数组 `nums` ，返回其中的多数元素。多数元素是指在数组中出现次数 **大于** `⌊ n/2 ⌋` 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

 

**示例 1：**

```
输入：nums = [3,2,3]
输出：3
```

**示例 2：**

```
输入：nums = [2,2,1,1,1,2,2]
输出：2
```

 

**提示：**

- `n == nums.length`
- `1 <= n <= 5 * 10^4`
- `-10^9 <= nums[i] <= 10^9`

 

**进阶：**尝试设计时间复杂度为 O(n)、空间复杂度为 O(1) 的算法解决此问题。



### 思路

建哈希表记录每个元素出现的次数，然后遍历哈希表找到出现次数**大于** `⌊ n/2 ⌋` 的元素，时间复杂度 `O(n)` ，空间复杂度 `O(n)` 。

代码如下：

```c++
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int n = nums.size();
        unordered_map<int, int> f;
        for (const auto& num: nums) {
            if (f.find(num) != f.end()) {
                ++f[num];
            } else {
                f.insert({num, 1});
            }
        }
        int res = 0;
        for (const auto& i: f) {
            if (i.second > n / 2) {
                res = i.first;
                break;
            }
        }
        return res;
    }
};
```

### Boyer-Moore 投票算法

该算法用于**快速找出数组中出现次数超过一半的元素**。

假设有一个数组，例如 `[2, 2, 3, 2, 4, 2, 5]`，其中元素 `2` 出现了 4 次（超过半数）。如何用 **O(n)时间、O(1)空间** 找出这个“多数元素”？

Boyer-Moore 投票算法的核心思想是**消消乐**，想象一群不同阵营的人混战，每次让两个不同阵营的人“同归于尽”。最终剩下的阵营一定是人数超过一半的阵营（如果存在的话）。

具体步骤：

1. 初始化
   - 维护一个候选值 `candidate` 和一个计数器 `count`
   - 初始时 `candidate` 为空，`count = 0`
2. 遍历数组
   - 如果 `count == 0` ，将当前遍历到的元素设置成新的 `candidate` ，并设 `count = 1`
   - 如果当前遍历到的元素等于 `candidate` ，那么令 `count` + 1
   - 如果当前遍历到的元素不等于 `candidate` ，那么令 `count` - 1
3. 遍历数组结束后，`candidate` 的值即为“多数元素”

**Code**

```c++
#include<bits/stdc++.h>
using namespace std;

int solve(vector<int>& nums) {
    int candidate = 0;
    int cnt = 0;
    for (int& x : nums) {
        if (cnt == 0) {
            candidate = x;
        }
        cnt += x == candidate ? 1 : -1;
    }
    return candidate;
}

int main() {
    vector<int> nums1 = {3,2,3};
    vector<int> nums2 = {2,2,1,1,1,2,2,};
    cout << solve(nums1) << endl;
    cout << solve(nums2) << endl;
    return 0;
}
```







## 98.颜色分类

给定一个包含红色、白色和蓝色、共 `n` 个元素的数组 `nums` ，**[原地](https://baike.baidu.com/item/原地算法)** 对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

我们使用整数 `0`、 `1` 和 `2` 分别表示红色、白色和蓝色。

必须在不使用库内置的 sort 函数的情况下解决这个问题。

 

**示例 1：**

```
输入：nums = [2,0,2,1,1,0]
输出：[0,0,1,1,2,2]
```

**示例 2：**

```
输入：nums = [2,0,1]
输出：[0,1,2]
```

 

**提示：**

- `n == nums.length`
- `1 <= n <= 300`
- `nums[i]` 为 `0`、`1` 或 `2`

 

**进阶：**

- 你能想出一个仅使用常数空间的一趟扫描算法吗？



### 思路

可以直接调用 `sort` 函数

```c++
class Solution {
public:
    void sortColors(vector<int>& nums) {
        sort(nums.begin(), nums.end());
    }
};
```

不用 `sort` 时，可以用**单指针**法进行原地排序：

首先令 `ptr` 指向 `0...index1` ，表示 0 所在的区间

- 初始化将 `ptr` 设置成 0 ，遍历一次数组，如果遇到 0 ，就交换 `nums[ptr]` 和 `nums[i]` ；
- 从 `ptr` 的位置开始，再遍历一次数组，如果遇到 1 ，就交换 `nums[ptr]` 和 `nums[i]` 

```c++
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int n = nums.size();
        int ptr = 0;
        for (int i = 0; i < n; ++i) {
            if (nums[i] == 0) {
                swap(nums[i], nums[ptr]);
                ++ptr;
            }
        }
        for (int i = ptr; i < n; ++i) {
            if (nums[i] == 1) {
                swap(nums[i], nums[ptr]);
                ++ptr;
            }
        }
    }
};
```

上面的单指针需要遍历两次数组，我们也可以使用双指针只遍历一次数组：

指针 `p0` 用来交换 0，`p1` 用来交换 1，两个指针初始值均为 0。当我们从左向右遍历整个数组时：

- 遇到 0 时，将当前元素与 `p0` 位置交换。此时，如果 `p0 < p1` ，说明原 `p0` 位置是 1（`p1` 总在 `p0` 之后），交换后当前元素会变成 1，需要再次将其与 `p1` 位置的元素交换，确保 1 被移动到中间区域，随后同时递增 `p1` 和 `p0` 。
- 遇到 1 时，直接与 `p1` 位置交换，递增 `p1` 。

```c++
#include<bits/stdc++.h>
using namespace std;

void solve(vector<int>& nums) {
    int p0 = 0, p1 = 0;
    for (int i = 0; i < nums.size(); ++i) {
        if (nums[i] == 1) {
            swap(nums[i], nums[p1]);
            ++p1;
        } else if (nums[i] == 0) {
            swap(nums[i], nums[p0]);
            if (p0 < p1) {
                swap(nums[i], nums[p1]);
            }
            ++p0;
            ++p1;
        }
    }
}

int main() {
    vector<int> nums = {2,0,2,1,1,0};
    solve(nums);
    for (int& x : nums) cout << x << " ";
    return 0;
}
```



## 99.下一个排列

整数数组的一个 **排列** 就是将其所有成员以序列或线性顺序排列。

- 例如，`arr = [1,2,3]` ，以下这些都可以视作 `arr` 的排列：`[1,2,3]`、`[1,3,2]`、`[3,1,2]`、`[2,3,1]` 。

整数数组的 **下一个排列** 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 **下一个排列** 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。

- 例如，`arr = [1,2,3]` 的下一个排列是 `[1,3,2]` 。
- 类似地，`arr = [2,3,1]` 的下一个排列是 `[3,1,2]` 。
- 而 `arr = [3,2,1]` 的下一个排列是 `[1,2,3]` ，因为 `[3,2,1]` 不存在一个字典序更大的排列。

给你一个整数数组 `nums` ，找出 `nums` 的下一个排列。

必须**[ 原地 ](https://baike.baidu.com/item/原地算法)**修改，只允许使用额外常数空间。

 

**示例 1：**

```
输入：nums = [1,2,3]
输出：[1,3,2]
```

**示例 2：**

```
输入：nums = [3,2,1]
输出：[1,2,3]
```

**示例 3：**

```
输入：nums = [1,1,5]
输出：[1,5,1]
```

 

**提示：**

- `1 <= nums.length <= 100`
- `0 <= nums[i] <= 100`





### 思路

STL 中有求下一个排列的函数：`next_permutation()` ，可以直接调用

```c++
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        next_permutation(nums.begin(), nums.end());
    }
};
```

手写如下

注意到下一个排列总是比当前排列大，除非该排列已经是最大的排列。我们希望找到一种方法，能够找到一个大于当前序列的新序列，且变大的幅度尽可能小。具体地：

1. 我们需要将一个左边的**较小数**与一个右边的**较大数**交换，以能够让当前排列变大，从而得到下一个排列。
2. 同时我们要让这个**较小数尽量靠右，而较大数尽可能小**。当交换完成后，较大数右边的数需要升序重新排列。这样可以保证新排列大于原来排列的情况下，**使变大的幅度尽可能小**。

以 `[4,5,2,6,3,1]` 为例：

- 我们能找到的符合条件的一对**较小数**与**较大数**的组合为 2 与 3 ，**满足较小数尽量靠右，而较大数尽可能小**。
- 当我们完成交换后，排列变为：`[4,5,3,6,2,1]` ，此时我们重排**较小数**右边的序列，序列变为：`[4,5,3,1,2,6]` 。

具体的，对于长度为 `n` 的排列 `a` ：

1. 首先从后向前查找第一个顺序对 `(i, i+1)` ，满足 `a[i] < a[i+1]` 。这样**较小数**就是 `a[i]` 。此时 `[i+1, n)` 必然是下降序列。
2. 如果找到了顺序对，那么在区间 `[i+1, n)` 中从后向前查找第一个元素 `j` 满足 `a[i] < a[j]` 。这样**较大数**即为 `a[j]` 。
3. 交换 `a[i]` 与 `a[j]` ，此时可以证明区间 `[i+1, n)` 必为降序。我们可以直接用双指针反转区间 `[i+1, n)` 使其变为升序，而无需对该区间进行排序。

注：如果在步骤 1 找不到顺序对，**说明当前序列已经是一个降序序列，即最大的序列**，我们直接跳过步骤 2 执行步骤 3，即可得到最小的升序序列。

代码如下：

```c++
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int i = nums.size() - 2;
        while (i >= 0 && nums[i] >= nums[i+1]) {
            --i;
        }
        if (i >= 0) {
            int j = nums.size() - 1;
            while (j >= 0 && nums[i] >= nums[j]) {
                --j;
            }
            swap(nums[i], nums[j]);
        }
        reverse(nums.begin() + i + 1, nums.end());
    }
};
```





## 100.寻找重复数

给定一个包含 `n + 1` 个整数的数组 `nums` ，其数字都在 `[1, n]` 范围内（包括 `1` 和 `n`），可知至少存在一个重复的整数。

假设 `nums` 只有 **一个重复的整数** ，返回 **这个重复的数** 。

你设计的解决方案必须 **不修改** 数组 `nums` 且只用常量级 `O(1)` 的额外空间。

 

**示例 1：**

```
输入：nums = [1,3,4,2,2]
输出：2
```

**示例 2：**

```
输入：nums = [3,1,3,4,2]
输出：3
```

**示例 3 :**

```
输入：nums = [3,3,3,3,3]
输出：3
```

 

 

**提示：**

- `1 <= n <= 10^5`
- `nums.length == n + 1`
- `1 <= nums[i] <= n`
- `nums` 中 **只有一个整数** 出现 **两次或多次** ，其余整数均只出现 **一次**

 

**进阶：**

- 如何证明 `nums` 中至少存在一个重复的数字?
- 你可以设计一个线性级时间复杂度 `O(n)` 的解决方案吗？



### 思路

排序后遍历

```c++
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        for (int i = 0; i < nums.size() - 1; ++i) {
            if (nums[i] == nums[i+1]) {
                return nums[i];
            }
        }
        return 0;
    }
};
```





## 补充：LFU 缓存

请你为 [最不经常使用（LFU）](https://baike.baidu.com/item/缓存算法)缓存算法设计并实现数据结构。

实现 `LFUCache` 类：

- `LFUCache(int capacity)` - 用数据结构的容量 `capacity` 初始化对象
- `int get(int key)` - 如果键 `key` 存在于缓存中，则获取键的值，否则返回 `-1` 。
- `void put(int key, int value)` - 如果键 `key` 已存在，则变更其值；如果键不存在，请插入键值对。当缓存达到其容量 `capacity` 时，则应该在插入新项之前，移除最不经常使用的项。在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，应该去除 **最久未使用** 的键。

为了确定最不常使用的键，可以为缓存中的每个键维护一个 **使用计数器** 。使用计数最小的键是最久未使用的键。

当一个键首次插入到缓存中时，它的使用计数器被设置为 `1` (由于 put 操作)。对缓存中的键执行 `get` 或 `put` 操作，使用计数器的值将会递增。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

 

**示例：**

```
输入：
["LFUCache", "put", "put", "get", "put", "get", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [3], [4, 4], [1], [3], [4]]
输出：
[null, null, null, 1, null, -1, 3, null, -1, 3, 4]

解释：
// cnt(x) = 键 x 的使用计数
// cache=[] 将显示最后一次使用的顺序（最左边的元素是最近的）
LFUCache lfu = new LFUCache(2);
lfu.put(1, 1);   // cache=[1,_], cnt(1)=1
lfu.put(2, 2);   // cache=[2,1], cnt(2)=1, cnt(1)=1
lfu.get(1);      // 返回 1
                 // cache=[1,2], cnt(2)=1, cnt(1)=2
lfu.put(3, 3);   // 去除键 2 ，因为 cnt(2)=1 ，使用计数最小
                 // cache=[3,1], cnt(3)=1, cnt(1)=2
lfu.get(2);      // 返回 -1（未找到）
lfu.get(3);      // 返回 3
                 // cache=[3,1], cnt(3)=2, cnt(1)=2
lfu.put(4, 4);   // 去除键 1 ，1 和 3 的 cnt 相同，但 1 最久未使用
                 // cache=[4,3], cnt(4)=1, cnt(3)=2
lfu.get(1);      // 返回 -1（未找到）
lfu.get(3);      // 返回 3
                 // cache=[3,4], cnt(4)=1, cnt(3)=3
lfu.get(4);      // 返回 4
                 // cache=[3,4], cnt(4)=2, cnt(3)=3
```

 

**提示：**

- `1 <= capacity <= 10^4`
- `0 <= key <= 10^5`
- `0 <= value <= 10^9`
- 最多调用 `2 * 10^5` 次 `get` 和 `put` 方法



### 思路

![460-2-c.png](https://pic.leetcode.cn/1695621293-JySfYQ-460-2-c.png)

**Code**

```c++
class Node {
public:
    int key;
    int value;
    int freq = 1;
    Node* prev;
    Node* next;

    Node(int k = 0, int v = 0) : key(k), value(v) {}
};

class LFUCache {
private:
    int min_freq;
    int capacity;
    // 链表中找到对应的书
    unordered_map<int, Node*> key_to_node;
    // 对应几摞书，键是看书的次数
    unordered_map<int, Node*> freq_to_dummy;

    Node* get_node(int key) {
        auto it = key_to_node.find(key);
        if (it == key_to_node.end()) { // 没有这本书
            return nullptr;
        }
        Node* node = it->second; // 有这本书
        remove(node); // 抽出这本书
        Node* dummy = freq_to_dummy[node->freq]; // 获取 node 所在的书堆
        if (dummy->prev == dummy) { // 抽出来后，这摞书是空的
            freq_to_dummy.erase(node->freq); // 移除空链表
            delete dummy;
            // 如果这摞书在最左边，最小频率 +1
            if (min_freq == node->freq) {
                ++min_freq;
            }
        }
        push_front(++node->freq, node); // 放在右边这摞书的最上面
        return node;
    }

    // 创建新的双向链表（创建新的书堆）
    Node* new_list() {
        Node* dummy = new Node();
        dummy->prev = dummy;
        dummy->next = dummy;
        return dummy;
    }

    // 在链表头添加一个节点（把一本书放在最上面）
    void push_front(int freq, Node* x) {
        auto it = freq_to_dummy.find(freq);
        if (it == freq_to_dummy.end()) { // 这摞书为空
            // 创建一个新双向链表，并将其插入到哈希表中
            // emplace 返回的是 pair 对象，first 是指向插入的元素的迭代器，second 是布尔值，表示是否插入成功
            it = freq_to_dummy.emplace(freq, new_list()).first;
        }
        Node* dummy = it->second;
        x->prev = dummy;
        x->next = dummy->next;
        x->prev->next = x;
        x->next->prev = x;
    }

    // 删除一个节点（抽出一本书）
    void remove(Node* x) {
        x->prev->next = x->next;
        x->next->prev = x->prev;
    }

public:
    LFUCache(int capacity) : capacity(capacity){}
    
    int get(int key) {
        Node* node = get_node(key);
        return node ? node->value : -1;
    }
    
    void put(int key, int value) {
        Node* node = get_node(key);
        if (node) { // 有这本书
            node->value = value; // 更新 value
            return;
        }
        if (key_to_node.size() == capacity) { // 书太多了
            Node* dummy = freq_to_dummy[min_freq]; // 找到最少使用的那摞书
            Node* back_node = dummy->prev; // 最少使用的那摞书的最下面的书
            key_to_node.erase(back_node->key);
            remove(back_node); // 移除
            delete back_node;
            if (dummy->prev == dummy) { // 这摞书是空的
                freq_to_dummy.erase(min_freq); // 移除空链表
                delete dummy;
            }
        }
        key_to_node[key] = node = new Node(key, value); // 新书
        push_front(1, node); // 放在 看过一次 的最上面
        min_freq = 1;
    }
};

/**
 * Your LFUCache object will be instantiated and called as such:
 * LFUCache* obj = new LFUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```

