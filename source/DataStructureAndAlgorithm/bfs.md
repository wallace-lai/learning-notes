# 广度优先搜索

作者：wallace-lai </br>
发布：2024-02-16 </br>
更新：2024-02-16 <br>

## 基础知识
这里先给出广度优先算法BFS的算法的二叉树模板：

```cpp
void BFS(TreeNode *root)
{
    // 1. push root node as search start
    queue<TreeNode*> q;
    q.push(root);

    // 2. search process
    while (!q.empty()) {
        // 3. len is node nunber for current level
        size_t len = q.size();
        for (size_t i = 0; i < len; i++) {
            // 4. fetch one current node
            TreeNode *curr = q.front();
            q.pop();

            // 5. do something on curr TreeNode

            // 6. push children of curr TreeNode back
            if (curr->left != nullptr) {
                q.push(curr->left);
            }
            if (curr->right != nullptr) {
                q.push(curr->right);
            }
        }
    }
}
```

如果对上述模板所描述的BFS算法非常熟悉，那么你可以很容易地对上述模板进行扩展以适应不同的场景。如下所示：

（1）针对多叉树，可能第6部得修改成将当前结点的所有孩子结点加入队列中

```cpp
for (auto &x : curr->_children) {
    q.push(x);
}
```

（2）针对需要禁止重复访问同一个结点的场景，同样在第6部，你需要只加入还未被访问过的结点
```cpp
for (auto &x : curr->_children) {
    if (!vis[x]) {
        q.push(x);
    }
}
```

（3）针对网格图的场景，同样也是第6部，在生成下一层次的子结点时可能有不一样的方式
```cpp
int x = curr->x;
int y = curr->y;
for (int k = 0; k < 4; k++) {
    int nx = x + dx[k];
    int ny = y + dy[k];
    if (0 <= nx && nx < row && 0 <= ny && ny < col && !vis[nx][ny]) {
        // do something
        vis[nx][ny] = true;
        q.push(make_pair(nx, ny));
    }
}
```

总之，上述模板中可以变换的地方还是比较多的。

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


## LCP 0007 传递信息【简单】
[链接](https://leetcode.cn/problems/chuan-di-xin-xi/description/)

题目不难，直接运用BFS模板即可。但需要注意几点：

（1）题目要求是必须在第k轮传递的时候刚好传递到`n-1`的位置

（3）所有大于k轮的搜索都没有意义，必须要利用这点停止继续搜索

## LCP 0004 开幕式焰火【简单】
[链接](https://leetcode.cn/problems/sZ59z6/description/)

比较简单，直接运用BFS模板即可