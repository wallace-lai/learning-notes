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

## LeetCode 0365 水壶问题
[链接](https://leetcode.cn/problems/water-and-jug-problem/description/)

这道题非常有意思，看起来和BFS没关系，但竟然也能用BFS来解决。

取两水壶的水量x和y构成一个状态(x, y)，只要经过允许的三种操作后能得到以下的状态就表示可以使用这两个水壶得到既定的水量t

（1）(t, 0)

（2）(0, t)

（3）(x, y) 且 x + y = t

允许的操作共有三种，分别是：

（1）装满任意一个水壶

（2）清空任意一个水壶

（3）从一个水壶向另外一个水壶倒水，直到装满或者倒空

对应的状态转移公式如下所示，其中Cx和Cy分别表示两个水壶的容量

```
(x, y)
    // 1. 清空一个水壶
    --> (0, y)  x != 0
    --> (x, 0)  y != 0

    // 2. 装满一个水壶
    --> (Cx, y) x < Cx
    --> (x, Cy) y < Cy

    // 3. 一个水壶向另一个水壶倒水
    --> (x - (Cy - y), Cy)  x >= Cy - y
    --> (0, y + x)          x < Cy - y
    --> (Cx, y - (Cx - x))  y >= Cx - x
    --> (x + y, 0)          y < Cx - x
```

以上八种情况就是BFS搜索时扩展子结点的依据，写成代码就是：

```cpp
class Solution {
private:
    void ProcessChild(int nx, int ny, queue<pair<int, int>> &q, unordered_set<string> &us) {
        string status = to_string(nx) + "*" + to_string(ny);
        if (us.count(status) == 0) {
            us.insert(status);
            q.push(make_pair(nx, ny));
        }
    }

public:
    bool canMeasureWater(int jug1Capacity, int jug2Capacity, int targetCapacity) {
        if (jug1Capacity + jug2Capacity < targetCapacity) {
            return false;
        }

        queue<pair<int, int>> q;
        q.push(make_pair(0, 0));
        unordered_set<string> us;
        us.insert("0*0");

        while (!q.empty()) {
            size_t len = q.size();
            for (size_t i = 0; i < len; i++) {
                pair<int, int> curr = q.front();
                q.pop();

                int x = curr.first;
                int y = curr.second;
                int nx, ny;
                if (x != 0) {
                    nx = 0;
                    ny = y;
                    if (ny == targetCapacity) {
                        return true;
                    }
                    ProcessChild(nx, ny, q, us);
                }
                if (y != 0) {
                    nx = x;
                    ny = 0;
                    if (nx == targetCapacity) {
                        return true;
                    }
                    ProcessChild(nx, ny, q, us);
                }
                if (x < jug1Capacity) {
                    nx = jug1Capacity;
                    ny = y;
                    if (nx + ny == targetCapacity) {
                        return true;
                    }
                    ProcessChild(nx, ny, q, us);
                }
                if (y < jug2Capacity) {
                    nx = x;
                    ny = jug2Capacity;
                    if (nx + ny == targetCapacity) {
                        return true;
                    }
                    ProcessChild(nx, ny, q, us);
                }
                if (x >= jug2Capacity - y) {
                    nx = x - (jug2Capacity - y);
                    ny = jug2Capacity;
                    if (nx + ny == targetCapacity) {
                        return true;
                    }
                    ProcessChild(nx, ny, q, us);
                }
                if (x < jug2Capacity - y) {
                    nx = 0;
                    ny = y + x;
                    if (ny == targetCapacity) {
                        return true;
                    }
                    ProcessChild(nx, ny, q, us);
                }
                if (y >= jug1Capacity - x) {
                    nx = jug1Capacity;
                    ny = y - (jug1Capacity - x);
                    if (nx + ny == targetCapacity) {
                        return true;
                    }
                    ProcessChild(nx, ny, q, us);
                }
                if (y < jug1Capacity - x) {
                    nx = x + y;
                    ny = 0;
                    if (nx == targetCapacity) {
                        return true;
                    }
                    ProcessChild(nx, ny, q, us);
                }
            }
        }

        return false;
    }
};
```

但是上面的代码是会超时的，只要你不使用贝祖定理，这就很狗血、很无聊了。

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