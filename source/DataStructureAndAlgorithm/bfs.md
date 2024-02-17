# 广度优先搜索

作者：wallace-lai </br>
发布：2024-02-16 </br>
更新：2024-02-16 <br>

## LeetCode 0310 最小高度树
[链接](https://leetcode.cn/problems/minimum-height-trees/description/)


## LeetCode 0429 N叉树的层序遍历
[链接](https://leetcode.cn/problems/n-ary-tree-level-order-traversal/description/)

非常简单和经典的一个广度优先搜索案例，直接把广度优先搜索的模板套上去即可。

```cpp
vector<vector<int>> levelOrder(Node* root) {
    vector<vector<int>> result;
    if (root == nullptr) {
        return result;
    }

    queue<Node*> q;
    q.push(root);
    while (!q.empty()) {
        vector<int> level;

        size_t size = q.size();
        for (size_t i = 0; i < size; i++) {
            Node *curr = q.front();
            q.pop();

            level.push_back(curr->val);
            vector<Node*> &child = curr->children;
            for (size_t x = 0; x < child.size(); x++) {
                q.push(child[x]);
            }
        }

        result.push_back(level);
    }

    return result;
}
```