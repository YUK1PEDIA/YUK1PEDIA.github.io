---
layout: post
title: LCR-二叉树
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







## 2.相同的树

给你两棵二叉树的根节点 `p` 和 `q` ，编写一个函数来检验这两棵树是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/12/20/ex1.jpg)

```
输入：p = [1,2,3], q = [1,2,3]
输出：true
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/12/20/ex2.jpg)

```
输入：p = [1,2], q = [1,null,2]
输出：false
```

**示例 3：**

![img](https://assets.leetcode.com/uploads/2020/12/20/ex3.jpg)

```
输入：p = [1,2,1], q = [1,1,2]
输出：false
```

 

**提示：**

- 两棵树上的节点数目都在范围 `[0, 100]` 内
- `-10^4 <= Node.val <= 10^4`





**思路**

解决二叉树问题就是要灵活运用递归的性质。我们需要思考，如何将原问题分解为更小的子问题来解决，比如这道题。

要判断两棵二叉树是否相等，我们可以将它分解为以下子问题：

- 两棵二叉树的根节点值是否相等？
- 两个二叉树根节点的左边两棵子树是否相同？右边两棵子树是否相同？

分解完子问题后，我们便可以得出判断方法了：**先判断根节点的值是否相等，再判断根节点的左右子树是否相等**。

我们怎么判断根节点的左右子树是否相等呢？这个问题可以用递归解决。

这里需要判断出递归的边界条件：当递归到某个节点为空，就需要判断另外一个节点是否为空，如果为空，返回 *true* ，否则返回 *false*。



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
    bool isSameTree(TreeNode* p, TreeNode* q) {
        if (p == nullptr || q == nullptr) return p == q;
        return p->val == q->val && isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
    }
};
```





## 3.对称二叉树

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





**思路**

判断二叉树是否对称，同样需要将其分解为更小的子问题：

- 首先，根节点已经是对称的，可以不用管
- 然后需要判断根节点的左右子树是否对称

于是我们可以这样做：

1. **把给定的树拆成左右两棵树**，看根节点是否相同。
2. 然后递归左边的左子树，和右边的右子树；以及左边的右子树和右边的左子树，看他们是否相等

这里可以直接用 *T2* 的代码：



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
    bool isSame(TreeNode* p, TreeNode* q) {
        if (p == nullptr || q == nullptr) return p == q;
        return p->val == q->val && isSame(p->left, q->right) && isSame(p->right, q->left);
    }

    bool isSymmetric(TreeNode* root) {
        return isSame(root->left, root->right);
    }
};
```







## 4.平衡二叉树

给定一个二叉树，判断它是否是平衡二叉树

 



 

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/06/balance_1.jpg)

```
输入：root = [3,9,20,null,null,15,7]
输出：true
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2020/10/06/balance_2.jpg)

```
输入：root = [1,2,2,3,3,null,null,4,4]
输出：false
```

**示例 3：**

```
输入：root = []
输出：true
```

 

**提示：**

- 树中的节点数在范围 `[0, 5000]` 内
- `-10^4 <= Node.val <= 10^4`





**思路**

要判断一棵树是否是平衡二叉树，我们需要计算这棵树的左右子树高度，再判断左右子树高度差是否大于 *1* 。

那么我们**如何通过计算左右子树的高度来判断平衡树呢**？

由于树的高度都是非负数，我们可以利用负数来表示当前子树不平衡。如果当前子树不平衡，那么包含这个子树的树也一定不平衡，直接返回即可。



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
    int get_height(TreeNode* p) {
        if (p == nullptr) return 0;
        // 递归计算左子树高度
        int left_height = get_height(p->left);
        // 高度为 -1 表示当前树不平衡，那么包含当前树的树也一定不平衡
        if (left_height == -1) return -1;
        // 递归计算右子树高度
        int right_height = get_height(p->right);
        if (right_height == -1 || abs(left_height - right_height) > 1) return -1;
        return max(left_height, right_height) + 1;
    }

    bool isBalanced(TreeNode* root) {
        // 如果不为 -1 ，则代表是平衡二叉树
        return get_height(root) != -1;
    }
};
```







## 5.二叉树的右视图

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





**思路**

要求二叉树的右视图数组，除了 *BFS* 还可以这样递归：

- 先不断递归二叉树的右子树，将最右侧的数字都加入到答案数组 *res* 中
- 然后递归二叉树的左子树，用一个变量 *depth* 记录当前递归深度，如果 *depth* 超出了 *res* 的长度，**说明当前”左边的“节点没有被最右边的节点遮挡**，那么将其添加到答案数组中即可



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
private:
    vector<int> res;

public:
    void dfs(TreeNode* root, int depth) {
        if (root == nullptr) return; // 递归边界
        if (depth == res.size()) res.push_back(root->val);
        dfs(root->right, depth + 1);
        dfs(root->left, depth + 1);
    }

    vector<int> rightSideView(TreeNode* root) {
        dfs(root, 0);
        return res;
    }
};
```







## 6.子结构判断

给定两棵二叉树 `tree1` 和 `tree2`，判断 `tree2` 是否以 `tree1` 的某个节点为根的子树具有 **相同的结构和节点值** 。
注意，**空树** 不会是以 `tree1` 的某个节点为根的子树具有 **相同的结构和节点值** 。

 

**示例 1：**

 

![img](https://pic.leetcode.cn/1694684670-vwyIgY-two_tree.png)

 

```
输入：tree1 = [1,7,5], tree2 = [6,1]
输出：false
解释：tree2 与 tree1 的一个子树没有相同的结构和节点值。
```

**示例 2：**

![img](https://pic.leetcode.cn/1694685602-myWXCv-two_tree_2.png)

```
输入：tree1 = [3,6,7,1,8], tree2 = [6,1]
输出：true
解释：tree2 与 tree1 的一个子树拥有相同的结构和节点值。即 6 - > 1。
```

 

**提示：**

```
0 <= 节点个数 <= 10000
```





**思路**

如果树 *B* 是树 *A* 的子结构，则子结构的根节点可能为树 *A* 的任意一个节点。因此，判断树 *B* 是否为树 *A* 的子结构，需要完成以下两步工作：

1. 先序遍历树 *A* 中的每个节点 *node* （调用函数 `isSubStructure(A, B)` 遍历树 *A* ）
2. 判断树 *A* 中以 *node* 为根节点的子树是否包含树 *B* （调用函数 `recur(A, B)` ，判定树 *A* 包含树 *B* ）



`recur(A, B)` 用于进行 *A* 树和 *B* 树的匹配，判断树 *A* 是否包含树 *B* （**此处的”包含“建立在树 *A* 和树 *B* 的根节点相同的条件上**）

1. **终止条件：**
   1. 当节点 `B` 为空：说明树 `B` 已匹配完成（越过叶子节点），因此返回 *true* ；
   2. 当节点 `A` 为空：说明已经越过树 `A` 的叶节点，即匹配失败，返回 *false* ；
   3. 当节点 `A` 和 `B` 的值不同：说明匹配失败，返回 *false* ；
2. **返回值：**
   1. 判断 `A` 和 `B` 的 **左子节点** 是否相等，即 `recur(A.left, B.left)` ；
   2. 判断 `A` 和 `B` 的 **右子节点** 是否相等，即 `recur(A.right, B.right)` ；



**Code**

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
    bool recur(TreeNode* A, TreeNode* B) {
        // A 树和 B 树从根节点开始同步往下递归
        if(B == nullptr) return true; // B 先到达空节点，说明 A 树包含 B 树
        if(A == nullptr || A->val != B->val) return false;
        return recur(A->left, B->left) && recur(A->right, B->right);
    }

public:
    bool isSubStructure(TreeNode* A, TreeNode* B) {
        return (A != nullptr && B != nullptr) && (recur(A, B) || isSubStructure(A->left, B) || isSubStructure(A->right, B));
    }
};
```

