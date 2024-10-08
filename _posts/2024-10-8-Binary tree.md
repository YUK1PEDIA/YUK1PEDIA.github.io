---
layout: post
title: 二叉树
description: 记录
tag: 算法
---

## 1.根据序列构造二叉树

### 根据前序与中序遍历序列构造二叉树

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



**思路**

**前序遍历**：按照「根-左子树-右子树」的顺序遍历二叉树。

**中序遍历**：按照「左子树-根-右子树」的顺序遍历二叉树。

我们来看看示例 1 是怎么生成这棵二叉树的。

![lc105-c.png](https://pic.leetcode.cn/1707907886-ICkiSC-lc105-c.png)

**递归边界**：如果 *preorder* 的长度是 0，对应着空节点，返回空。



**Code**

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
        if (preorder.empty()) return nullptr; // 空节点

        // 在中序序列中找根节点，记录左子树序列的长度
        int left_size = ranges::find(inorder, preorder[0]) - inorder.begin();
        // 在前序序列中找左子树
        vector<int> pre1(preorder.begin() + 1, preorder.begin() + 1 + left_size);
        // 在前序序列中找右子树
        vector<int> pre2(preorder.begin() + 1 + left_size, preorder.end());
        // 中序遍历同样的操作
        vector<int> in1(inorder.begin(), inorder.begin() + left_size);
        vector<int> in2(inorder.begin() + 1 + left_size, inorder.end());
        // 递归
        TreeNode *left = buildTree(pre1, in1);
        TreeNode *right = buildTree(pre2, in2);
        return new TreeNode(preorder[0], left, right);
    }
};
```



