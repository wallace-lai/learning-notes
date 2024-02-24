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

## LeetCode 0513 找树左下角的值【中等】
[链接](https://leetcode.cn/problems/find-bottom-left-tree-value/description/)

比较简单，直接运用BFS模板稍加改动即可。

## LeetCode 0515 在每个树行中找最大值【中等】
[链接](https://leetcode.cn/problems/find-largest-value-in-each-tree-row/description/)

比较简单，直接运用BFS模板稍加改动即可。


## LeetCode 0623 在二叉树中增加一行【中等】
[链接](https://leetcode.cn/problems/add-one-row-to-tree/description/)

解题思路比较简单：在使用BFS进行层序遍历时，在遍历到指定层级时，对该层级的所有结点都新增一层子结点即可。

核心代码如下：

```cpp
TreeNode* addOneRow(TreeNode* root, int val, int depth) {
    if (depth == 1) {
        TreeNode *result = new TreeNode(val);
        result->left = root;
        return result;
    }

    int count = 1;
    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        count++;

        size_t len = q.size();
        for (size_t i = 0; i < len; i++) {
            TreeNode *curr = q.front();
            q.pop();

            // 新增一层
            if (count == depth) {
                TreeNode *nLeft = new TreeNode(val);
                TreeNode *nRight = new TreeNode(val);
                nLeft->left = curr->left;
                nRight->right = curr->right;
                curr->left = nLeft;
                curr->right = nRight;
                continue;
            }

            if (curr->left != nullptr) {
                q.push(curr->left);
            }
            if (curr->right != nullptr) {
                q.push(curr->right);
            }
        }

        if (count == depth) {
            break;
        }
    }

    return root;
}
```

## LeetCode 0802 找到最终的安全状态【中等】
[链接](https://leetcode.cn/problems/find-eventual-safe-states/description/)

这题稍微复杂了点，解题思路如下：

（1）首先寻找没有连出有向边的结点，即终端结点，终端结点天然是安全结点

（2）从终端结点开始进行BFS搜索，如果有个结点所有连出的有向边最终都指向现存的安全结点，则将这样的结点加入到安全结点列表中

（3）重复上述过程，直到所有的安全结点都被找到为止

核心代码如下：

```cpp
bool IsSafe(size_t id, vector<vector<int>> &graph, unordered_set<int> &safeNode) {
    vector<int> &child = graph[id];
    for (auto &x : child) {
        if (safeNode.count(x) == 0) {
            return false;
        }
    }

    return true;
}

vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
    // 构建反向映射图
    vector<vector<int>> rGraph;
    rGraph.resize(graph.size());
    for (size_t i = 0; i < graph.size(); i++) {
        vector<int> &g = graph[i];
        for (auto &x : g) {
            rGraph[x].push_back(i);
        }
    }

    // 寻找终端节点存入安全结点列表
    queue<int> q;
    unordered_set<int> safeNode;
    for (size_t i = 0; i < graph.size(); i++) {
        vector<int> &g = graph[i];
        if (g.size() == 0) {
            safeNode.insert(i);
            q.push(i);
        }
    }

    // 从终端结点开始进行BFS搜索，如果找到安全结点则加入队列中
    while (!q.empty()) {
        size_t len = q.size();
        for (size_t i = 0; i < len; i++) {
            int curr = q.front();
            q.pop();

            vector<int> &child = rGraph[curr];
            for (auto &x : child) {
                if (safeNode.count(x) == 0 && IsSafe(x, graph, safeNode)) {
                    safeNode.insert(x);
                    q.push(x);
                }
            }
        }
    }

    vector<int> result(safeNode.begin(), safeNode.end());
    sort(result.begin(), result.end());
    return result;
}
```

## LeetCode 0919 完全二叉树插入器【中等】
[链接](https://leetcode.cn/problems/complete-binary-tree-inserter/description/)

题目稍微有点复杂，核心思路是：维护一个保存二叉树每一层结点的表，结点在表中的下标为i，则它对应父结点在上一层表中的下标为i / 2。核心代码如下：

```cpp
struct CBT {
    size_t level;
    vector<vector<TreeNode*>> links;
};

class CBTInserter {
private:
    CBT cbt;

public:
    CBTInserter(TreeNode* root) {
        int depth = -1;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            depth++;
            cbt.links.push_back(vector<TreeNode*>());
            vector<TreeNode*> &link = cbt.links[depth];

            size_t len = q.size();
            for (size_t i = 0; i < len; i++) {
                TreeNode *curr = q.front();
                q.pop();

                link.push_back(curr);
                if (curr->left != nullptr) {
                    q.push(curr->left);
                }
                if (curr->right != nullptr) {
                    q.push(curr->right);
                }
            }
        }

        cbt.level = depth;
    }
    
    int insert(int val) {
        size_t linkNum = cbt.links.size();
        size_t l = cbt.level;
        while (l < linkNum) {
            vector<TreeNode*> &link = cbt.links[l];
            if (link.size() < (1 << l)) {
                break;
            }
            l++;
        }

        cbt.level = l;
        if (cbt.level == linkNum) {
            cbt.links.push_back(vector<TreeNode*>());
        }

        vector<TreeNode*> &link = cbt.links[cbt.level];
        TreeNode *nval = new TreeNode(val);
        link.push_back(nval);

        size_t nvalPos = link.size() - 1;
        size_t rootPos = nvalPos / 2;
        vector<TreeNode*> &rootLink = cbt.links[cbt.level - 1];
        TreeNode *rootNode = rootLink[rootPos];
        if (rootNode->left == nullptr) {
            rootNode->left = nval;
        } else {
            rootNode->right = nval;
        }

        return rootNode->val;
    }
    
    TreeNode* get_root() {
        return cbt.links[0][0];
    }
};
```

## LeetCode 0934 最短的桥【中等】
[链接](https://leetcode.cn/problems/shortest-bridge/description/)

题目不难，解题思路如下：

（1）首先通过DFS找到其中一个岛所占据的所有方格存入source中，同时将该岛方格中的1修改成0

（2）将source中的所有位置入队列作为BFS搜索的起始位置，开始进行BFS搜索，只要搜到一个值为1的方格就说明搜索到了第二个岛。返回此时的层次计数器值即可

核心代码如下：

```cpp
void FindIsland(vector<vector<int>> &grid, int x, int y,
    vector<pair<int, int>> &island, vector<vector<bool>> &vis) {
    int n = grid.size();
    static int dx[] = {-1, 1, 0, 0};
    static int dy[] = {0, 0, -1, 1};

    int nx, ny;
    for (size_t k = 0; k < 4; k++) {
        nx = x + dx[k];
        ny = y + dy[k];
        if (0 <= nx && nx < n && 0 <= ny && ny < n && grid[nx][ny] == 1 && !vis[nx][ny]) {
            island.push_back(make_pair(nx, ny));
            vis[nx][ny] = true;
            grid[nx][ny] = 0;
            FindIsland(grid, nx, ny, island, vis);
        }
    }
}

int shortestBridge(vector<vector<int>>& grid) {
    int n = grid.size();
    vector<vector<bool>> vis(n, vector<bool>(n, false));

    vector<pair<int, int>> source;
    for (size_t i = 0; i < n; i++) {
        for (size_t j = 0; j < n; j++) {
            if (grid[i][j] == 1) {
                source.push_back(make_pair(i, j));
                vis[i][j] = true;
                grid[i][j] = 0;
                FindIsland(grid, i, j, source, vis);
                goto FIND_SOURCE_END;
            }
        }
    }

FIND_SOURCE_END:
    int count = -1;
    queue<pair<int, int>> q;
    for (auto iter = source.begin(); iter != source.end(); iter++) {
        q.push(*iter);
    }

    static int nx, ny;
    static int dx[] = {-1, 1, 0, 0};
    static int dy[] = {0, 0, -1, 1};

    while (!q.empty()) {
        count++;
        size_t len = q.size();
        for (size_t i = 0; i < len; i++) {
            pair<int, int> curr = q.front();
            q.pop();

            for (size_t k = 0; k < 4; k++) {
                nx = curr.first + dx[k];
                ny = curr.second + dy[k];
                if (0 <= nx && nx < n && 0 <= ny && ny < n) {
                    if (grid[nx][ny] == 1) {
                        return count;
                    }

                    if (!vis[nx][ny]) {
                        vis[nx][ny] = true;
                        q.push(make_pair(nx, ny));
                    }
                }
            }
        }
    }

    return count;
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

## LeetCode 2583 二叉树中的第 K 大层和【中等】
[链接](https://leetcode.cn/problems/kth-largest-sum-in-a-binary-tree/description/)

题目比较简单，属于BFS模板的简单应用，但是需要注意整数溢出的问题。


## LCP 0007 传递信息【简单】
[链接](https://leetcode.cn/problems/chuan-di-xin-xi/description/)

题目不难，直接运用BFS模板即可。但需要注意几点：

（1）题目要求是必须在第k轮传递的时候刚好传递到`n-1`的位置

（3）所有大于k轮的搜索都没有意义，必须要利用这点停止继续搜索

## LCP 0004 开幕式焰火【简单】
[链接](https://leetcode.cn/problems/sZ59z6/description/)

比较简单，直接运用BFS模板即可

## LCP 0067 装饰树【中等】

[链接](https://leetcode.cn/problems/KnLfVT/description/)

题目非常简单，你只需要在BFS遍历过程中在非叶子结点下放加上对应的装饰结点即可。代码略

