# 栈和队列

作者：wallace-lai </br>
发布：2024-03-03 </br>
更新：2024-03-03 <br>

## LeetCode 0225 用队列实现栈【简单】
[链接](https://leetcode.cn/problems/implement-stack-using-queues/description)

解题思路：用第二个栈实现队列中元素的逆序

核心代码：

```cpp
void push(int x) {
    backup.push(x);
    while (!main.empty()) {
        backup.push(main.front());
        main.pop();
    }

    swap(main, backup);
}
```

将新元素放到`backup`中，然后将`main`中的元素放到`backup`的尾部，这样实现了逆序。最后再交换二者即可。

**扩展：只用一个队列实现**

用队列实现栈的核心在于**将队列的顺序序列转变成对应的逆序序列**。假设队列中已经有两个元素`1`和`2`，并且已经是逆序，这时要插入元素`3`且只能用一个队列，应该怎么办呢？很简单，**先将`3`放到队列末尾，再将前面的两个元素弹出再放到队列末尾即可**。这样也可以达到逆序的目的。

```
queue : 2 -- 1

queue : 2 -- 1 -- 3
        1 -- 3 -- 2
        3 -- 2 -- 1
```

核心代码如下：

```cpp
void push(int x) {
    size_t len = q.size();
    q.push(x);
    for (size_t i = 0; i < len; i++) {
        int curr = q.front();
        q.pop();
        q.push(curr);
    }
}
```

## LeetCode 0232 用栈实现队列【简单】

[链接](https://leetcode.cn/problems/implement-queue-using-stacks/description/)

解题思路：

用栈实现队列的核心在与如何借助第二个栈，将栈中的逆序元素给转换成顺序元素。可以按照下面的方法完成转换操作：

（1）入队列时，往主栈中push即可

（2）pop和peek时，需要先将主栈中的元素pop出来并重新push进备栈，随后pop或者peek备栈中的首元素即可

核心代码：

```cpp
void push(int x) {
    main.push(x);
}

int pop() {
    if (backup.empty()) {
        while (!main.empty()) {
            backup.push(main.top());
            main.pop();
        }
    }

    int ret = backup.top();
    backup.pop();
    return ret;
}

int peek() {
    if (backup.empty()) {
        while (!main.empty()) {
            backup.push(main.top());
            main.pop();
        }
    }

    return backup.top();
}
```