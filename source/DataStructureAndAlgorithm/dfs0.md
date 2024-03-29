# 深度优先搜索

作者：wallace-lai </br>
发布：2024-02-15 </br>
更新：2024-02-15 <br>

## LeetCode 0094 二叉树的中序遍历【简单】

[链接](https://leetcode.cn/problems/binary-tree-inorder-traversal/description/)

求二叉树中序遍历的递归法很简单，在这里不赘述。这里只探索迭代法，因为迭代法相对更困难一点。

![二叉树](../media/images/DataStructureAndAlgorithm/dfs0.png)

所谓中序遍历指的是以**左根右**的顺序去遍历二叉树中的结点。对上面这棵二叉树，当root指针指向根结点时，你会怎么去做中序遍历？是不是会按照以下的大概顺序去作遍历。

（1）不断往左子树的方向走，直到找到最左结点，输出该结点。

（2）向上回溯，输出最左结点的父结点。注意此时如果父节点的右子树不为空，需要对右子树根结点重复步骤（1）

如果借用栈来实现上述步骤，应该怎么做呢？

（1）首先应该有一个while循环，目的是不断地往左子树走并把遍历过的结点入栈

（2）随后将栈顶元素弹出将其值添加到遍历序列的尾部

（3）将指针设置为栈顶元素结点的右子树，目的是对右子树重复步骤（1）

最终代码实现如下所示：

```cpp
vector<int> result;

vector<int> inorderTraversal(TreeNode* root) {
    stack<TreeNode*> stk;
    TreeNode *curr = root;
    while (curr != nullptr || !stk.empty()) {
        while (curr != nullptr) {
            stk.push(curr);
            curr = curr->left;
        }

        curr = stk.top();
        stk.pop();
        result.push_back(curr->val);
        curr = curr->right;
    }

    return result;
}
```

## LeetCode 0105 从前序与中序遍历序列构造二叉树【中等】
[链接](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/)

二叉树的前序遍历形式为：

```
[ 根节点, [左子树的前序遍历结果], [右子树的前序遍历结果] ]
```

中序遍历形式为：

```
[ [左子树的中序遍历结果], 根节点, [右子树的中序遍历结果] ]
```

从前序与中序遍历序列中还原成原始二叉树的思路为：

（1）获取二叉树前序遍历序列中的第一个元素作为根结点

（2）然后去中序遍历序列中找到根结点，这样就将中序序列分成了左右子树两个部分

（3）递归地对左右子树重复上述步骤，直到把整棵树给构造出来

注意有个可以优化的点是：因为树结点值不重复，所以可以使用哈希表保存结点值在中序遍历序列中的位置，以便快速查找根结点在中序遍历序列中的位置。核心代码如下：

```cpp
TreeNode *BuildTree(vector<int> &pre, int preBeg, int preEnd, vector<int> &in, int inBeg, int inEnd) {
    TreeNode *root = nullptr;

    if (preEnd - preBeg >= 1) {
        root = new TreeNode(pre[preBeg]);
        int rootPos = pos[root->val];
        int leftSize = rootPos - inBeg;
        root->left = BuildTree(pre, preBeg + 1, preBeg + 1 + leftSize, in, inBeg, inBeg + leftSize);
        root->right = BuildTree(pre, preBeg + 1 + leftSize, preEnd, in, inBeg + leftSize + 1, inEnd);
    }

    return root;
}

TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
    for (size_t i = 0; i < inorder.size(); i++) {
        pos[inorder[i]] = i;
    }

    return BuildTree(preorder, 0, preorder.size(), inorder, 0, inorder.size());
}
```

## LeetCode 0106 从中序与后序遍历序列构造二叉树【中等】
[链接](https://leetcode.cn/problems/shortest-bridge/description/)

和上一题是类似的，仍然是递归地构建即可，核心代码如下：

```cpp
TreeNode *Build(vector<int> &post, int postBeg, int postEnd, vector<int> &in, int inBeg, int inEnd) {
    TreeNode *result = nullptr;

    if (postEnd - postBeg >= 1) {
        int root = post[postEnd - 1];
        int rootPos = pos[root];
        int leftSize = rootPos - inBeg;

        result = new TreeNode(root);
        result->left = Build(post, postBeg, postBeg + leftSize, in, inBeg, inBeg + leftSize);
        result->right = Build(post, postBeg + leftSize, postEnd - 1, in, inBeg + leftSize + 1, inEnd);
    }

    return result;
}
```

## LeetCode 0144 二叉树的前序遍历【简单】
[链接](https://leetcode.cn/problems/binary-tree-preorder-traversal/description/)

前序遍历指的是按照**根左右**的顺序遍历二叉树中的结点，前序遍历用迭代法实现相对比较简单，只需要在遍历根结点的时候将左右子树放入栈中作为下一步遍历的起始结点即可。其代码实现如下所示：

```cpp
vector<int> result;

vector<int> preorderTraversal(TreeNode* root) {
    if (root == nullptr) {
        return result;
    }

    stack<TreeNode*> stk;
    stk.push(root);
    TreeNode *curr;
    while (!stk.empty()) {
        curr = stk.top();
        stk.pop();
        result.push_back(curr->val);
        // 注意左子树应该在右子树之前被遍历
        // 按照后进先出的特性右子树应该先入栈
        if (curr->right != nullptr) {
            stk.push(curr->right);
        }
        if (curr->left != nullptr) {
            stk.push(curr->left);
        }
    }

    return result;
}
```

## LeetCode 0145 二叉树的后序遍历【简单】
[链接](https://leetcode.cn/problems/binary-tree-postorder-traversal/description/)

后序遍历指的是按照**左右根**的顺序遍历二叉树中的结点，观察前序和后序遍历的不同。如果将前序遍历进行逆序，那么得到的顺序是**右左根**，和后序遍历的差别在于左右的顺序不同而已。

由此得到后序遍历的迭代算法思路如下：借用前序遍历框架得到**根右左**的遍历顺序，然后再将结果逆序，由此得到后序遍历的结果。代码如下所示：

```cpp
vector<int> result;

vector<int> postorderTraversal(TreeNode* root) {
    if (root == nullptr) {
        return result;
    }

    stack<TreeNode*> stk;
    stk.push(root);
    TreeNode *curr;
    while (!stk.empty()) {
        curr = stk.top();
        stk.pop();
        result.push_back(curr->val);
        // 注意和前序遍历的区别
        if (curr->left != nullptr) {
            stk.push(curr->left);
        }
        if (curr->right != nullptr) {
            stk.push(curr->right);
        }
    }

    reverse(result.begin(), result.end());
    return result;
}
```

## LeetCode 0235 二叉搜索树的最近公共祖先【中等】
[链接](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/description/)

解题思路如下：

（1）从根结点开始遍历

（2）如果根结点的值大于p和q的值，说明p和q在根结点的左子树中，将遍历的当前的结点移动到根结点的左子树

（3）如果根结点的值小于q和q的值，说明p和q在根结点的右子树中，将遍历的当前的结点移动到根结点的右子树

（4）如果不满足要求（2）和（3）说明当前的结点就是p和q开始分岔的结点，当前结点就是p和q的最近公共祖先结点

核心代码如下：

```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    TreeNode *curr = root;

    while (true) {
        if (p->val < curr->val && q->val < curr->val) {
            curr = curr->left;
        } else if (p->val > curr->val && q->val > curr->val) {
            curr = curr->right;
        } else {
            break;
        }
    }

    return curr;
}
```

## LeetCode 0590 N叉树的后序遍历【简单】

[链接](https://leetcode.cn/problems/n-ary-tree-postorder-traversal/description/)

比较简单，观察可知我只需要遵循以下的两个规则就能得到N叉树的后序遍历：

（1）如果结点存在子树，则优先遍历子树

（2）同级别的子树之间按照从左到右的顺序进行层序遍历

关键代码如下：

```cpp
void DFS(Node *root, vector<int> &result) {
    if (root == nullptr) {
        return;
    }

    for (auto &x : root->children) {
        DFS(x, result);
    }
    result.push_back(root->val);
}
```

## LeetCode 0797 所有可能的路径【中等】

[链接](https://leetcode.cn/problems/all-paths-from-source-to-target/description/)

比较简单，直接DFS搜索即可

## LeetCode 0889 根据前序和后序遍历构造二叉树【中等】
[链接](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-postorder-traversal/description/)

已知前序和后序遍历序列的情况下，无法确定唯一的一棵二叉树，可能存在多种情况。如下所示：

前序遍历：

```
根 【左子树】 【右子树】
```

后序遍历：

```
【左子树】 【右子树】 根
```

我们无法得知左子树和右子树的边界在哪，所以可能存在多种符合条件的情况。但是，题目只要求我们找出其中一种即可。有一个思路是：对于前序和后序，同时从左子树起始位置开始遍历，只要二者遍历找到的左子树构成的集合是相等的，就当我们找到了左子树。然后根据这个递归地去构造二叉树即可。

核心代码如下：

```cpp
TreeNode* BuildTree(vector<int> &pre, int preBeg, int preEnd,
    vector<int> &post, int postBeg, int postEnd) {
    TreeNode *result = nullptr;
    if (preEnd - preBeg >= 1) {
        int root = pre[preBeg];
        result = new TreeNode(root);

        set<int> preLeft;
        set<int> postLeft;
        for (int i = 0; i + postBeg < postEnd - 1; i++) {
            preLeft.insert(pre[i + preBeg + 1]);
            postLeft.insert(post[i + postBeg]);
            // 只要找到相同的左子树集合，就开始递归往下构建
            if (preLeft == postLeft) {
                result->left = BuildTree(pre, preBeg + 1, preBeg + 1 + i + 1, post, postBeg, postBeg + i + 1);
                result->right = BuildTree(pre, preBeg + 1 + i + 1, preEnd, post, postBeg + i + 1, postEnd - 1);
                break;
            }
        }
    }

    return result;
}
```

## LeetCode 0938 二叉搜索树的范围和【简单】
[链接](https://leetcode.cn/problems/range-sum-of-bst/description/)

思路很简单：

（1）先用中序遍历得到一个有序序列

（2）遍历有序序列，将在`[low, high]`中的元素相加即可

## LeetCode 1022 从根到叶的二进制数之和【简单】
[链接](https://leetcode.cn/problems/sum-of-root-to-leaf-binary-numbers/description/)

**解题思路：**

利用DFS遍历得到的路径所代表的二进制数是按照数的最高位到最低位排列的，如下所示。

```
1 -- 0 -- 1
sum : 1 * 4 + 0 * 2 + 1 * 1 = 5
```

如果要在遍历的过程中进行求值，可以使用以下的公式。对于上述的路径，计算过程如下。

```
sum = (sum << 1) + root->val

sum = 0
sum = (0 << 1) + 1 = 1
sum = (1 << 1) + 0 = 2
sum = (2 << 1) + 1 = 5
```

**核心代码：**

```cpp
void DFS(TreeNode *root, int sum, int &result) {
    // leaf node
    if (root->left == nullptr && root->right == nullptr) {
        int newSum = (sum << 1) + root->val;
        result += newSum;
        return;
    }

    // non leaf node
    int newSum = (sum << 1) + root->val;
    if (root->left != nullptr) {
        DFS(root->left, newSum, result);
    }
    if (root->right != nullptr) {
        DFS(root->right, newSum, result);
    }
}
```

## LeetCode 1457 二叉树中的伪回文路径【中等】

[链接](https://leetcode.cn/problems/pseudo-palindromic-paths-in-a-binary-tree/description/)

这个题目比较简单，核心是先通过DFS遍历找到一条路径，再判断该路径是否为伪回文路径即可，代码略。

## LeetCode 2331 计算布尔二叉树的值【简单】
[链接](https://leetcode.cn/problems/evaluate-boolean-binary-tree/description/)

这个题目比较简单，直接运用简单的DFS遍历即可。

