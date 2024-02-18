# 深度优先搜索

作者：wallace-lai </br>
发布：2024-02-15 </br>
更新：2024-02-15 <br>

## LeetCode 0094 二叉树的中序遍历
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

## LeetCode 0144 二叉树的前序遍历
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

## LeetCode 0145 二叉树的后序遍历

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