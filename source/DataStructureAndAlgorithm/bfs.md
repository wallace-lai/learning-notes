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

## LeetCode 0965 单值二叉树【简单】
[链接](https://leetcode.cn/problems/univalued-binary-tree/description/)

非常简单，直接应用BFS模板即可。

## LeetCode 1379 找出克隆二叉树中的相同结点【简单】
[链接](https://leetcode.cn/problems/find-a-corresponding-node-of-a-binary-tree-in-a-clone-of-that-tree/description/)

因为树中没有值相同的结点，所以只需要对克隆树进行BFS遍历找到值和target值相等的结点返回即可。注意，如果树中存在值相同结点，那么需要对原始树和克隆树同时进行BFS遍历，找到和target指针对应的克隆树结点并返回。

## LeetCode 1971 寻找图中是否存在路径【简单】
[链接](https://leetcode.cn/problems/find-if-path-exists-in-graph/description/)

从源点开始进行BFS搜索即可，只要能达到目的点说明存在有效路径。



