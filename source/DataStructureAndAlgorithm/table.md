# 索引

作者：wallace-lai </br>
发布：2024-02-25 </br>
更新：2024-08-09 <br>

## 数组

### 1. 前缀和

- LeetCode 0303
- LeetCode 0304
- LeetCode 0724
- LeetCode 1314
- 剑指Offer 0013

### 2. 前缀和加哈希表

- LeetCode 0325
- LeetCode 0437
- LeetCode 0523
- LeetCode 0525
- LeetCode 0560
- LeetCode 0713
- LeetCode 1124
- LeetCode 1658
- 剑指Offer 0010
- 剑指Offer 0011

【pending】进度在此


### 3. 前缀积

- LeetCode 0238

## 双指针

### 0. 未分类
- LeetCode 1260
- LeetCode 0048
- LeetCode 0054
- LeetCode 0059
- LeetCode 0061
- 剑指Offer 0029
- 剑指Offer 0058

### 1. 在数组或链表中去重、原地删除某类元素
- LeetCode 0026
- LeetCode 0027
- LeetCode 0083
- LeetCode 0283

### 2. 滑动窗口

### 3. 二分查找

### 4. nSum问题
- LeetCode 0001
- LeetCode 0015
- LeetCode 0018
- LeetCode 0167

### 5. 反转数组

- LeetCode 0005
- LeetCode 0151
- LeetCode 0344

### 6. 合并有序数组或链表
- LeetCode 0021
- LeetCode 0088
- LeetCode 0360
- LeetCode 0977

## 二叉树

二叉树的解题模式有两大类：

（1）**遍历思维**：通过遍历一遍二叉树配合外部变量记录来实现；

（2）**分解思维**：定义一个递归函数，通过子问题（子树）的答案推导出原问题的答案；

二叉树的遍历框架：

```cpp
void Traverse(TreeNode *root)
{
    if (root == nullptr) {
        return;
    }

    // 前序位置
    Traverse(root->left);
    // 中序位置
    Traverse(root->right);
    // 后续位置
}
```

**前序位置**：前序位置的代码在刚刚进入一个二叉树结点的时候执行；

**后序位置**：后序位置的代码在即将要离开一个二叉树结点的时候执行；

**中序位置**：中序位置的代码在一个二叉树结点左子树都遍历完毕，即将开始遍历右子树的时候执行

将这两个概念扩展到数组和指针：

```cpp
// 迭代遍历数组
void Traverse(vector<int> &arr)
{
    for (int i = 0; i < arr.size(); i++) {
        // do something
    }
}

// 递归遍历数组
void Traverse(vector<int> &arr, int i)
{
    if (i == arr.size()) {
        return;
    }

    // 前序位置
    Traverse(arr, i + 1);
    // 后序位置
}
```

```cpp
struct ListNode {
    int val;
    ListNode *next;
};

// 迭代遍历
void Traverse(ListNode *head)
{
    for (ListNode *p = head; p != nullptr; p = p->next) {
        // do something
    }
}

// 递归遍历
void Traverse(ListNode *head)
{
    if (head == nullptr) {
        return;
    }

    // 前序位置
    Traverse(head->next);
    // 后序位置
}
```





二叉树的层序遍历：

```cpp
void LevelOrderTraverse(TreeNode *root)
{
    if (root == nullptr) {
        return;
    }

    queue<TreeNode *> q;
    q.push(root);
    while (!q.empty()) {
        TreeNode *curr = q.front();
        q.pop();

        // do something on curr

        if (curr->left != nullptr) {
            q.push(curr->left);
        }
        if (curr->right != nullptr) {
            q.push(curr->right);
        }
    }
}
```

如果需要通过层序遍历获知二叉树的深度，可以这么做：

```cpp
void LevelOrderTraverse(TreeNode *root)
{
    if (root == nullptr) {
        return;
    }

    queue<TreeNode *> q;
    q.push(root);
    int depth = 1;

    while (!q.empty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            TreeNode *curr = q.front();
            q.pop();

            // do something on curr

            if (curr->left != nullptr) {
                q.push(curr->left);
            }
            if (curr->right != nullptr) {
                q.push(curr->right);
            }
        }

        depth++;
    }
}
```


## 二叉搜索树







