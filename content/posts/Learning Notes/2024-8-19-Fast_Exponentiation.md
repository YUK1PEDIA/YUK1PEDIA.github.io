---
layout: post
title: 快速幂
description: 记录
tag: 算法
---

![LC50-1.png](https://pic.leetcode.cn/1722389981-SLFaTc-LC50-1.png) 





**算法流程如下**

![LC50-2.png](https://pic.leetcode.cn/1722389988-gsDjeK-LC50-2.png) 



**代码实现时，注意 *n*=−2^31 的情况，取反后 *n*=2^31 超出 int 最大值。可以转成 64 位 int 解决。**

```c++
class Solution {
public:
    double myPow(double x, int N) {
        double ans = 1;
        long long n = N;
        if (n < 0) { // x^-n = (1/x)^n
            n = -n;
            x = 1 / x;
        }
        while (n) { // 从低到高枚举 n 的每个比特位
            if (n & 1) { // 当前比特位是 1
                ans *= x; // 把 x 乘到 ans 中
            }
            x *= x; // x 自身平方
            n >>= 1; // n 右移，继续枚举下一个比特位
        }
        return ans;
    }
};
```

