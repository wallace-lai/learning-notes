# 链表

作者：wallace-lai </br>
发布：2024-02-25 </br>
更新：2024-02-25 <br>

## LeetCode 0143 重排链表【中等】
[链接](https://leetcode.cn/problems/reorder-list/description/)

解题思路如下：

（1）先通过快慢指针将链表分为左右两部分，对右边部分进行逆序

（2）对左边部分和逆序后的右半部分进行merge操作即可

核心代码如下：

```cpp
void reorderList(ListNode* head) {
    ListNode *mid = FindMiddle(head);

    ListNode *l1 = head;
    ListNode *l2 = reverseList(mid->next);
    mid->next = nullptr;

    ListNode dummy(0, nullptr);
    ListNode *curr = &dummy;
    while (l1 != nullptr && l2 != nullptr) {
        curr->next = l1;
        l1 = l1->next;
        curr = curr->next;

        curr->next = l2;
        l2 = l2->next;
        curr = curr->next;
    }
    curr->next = l1;

    return;
}
```

## LeetCode 0146 LRU缓存【中等】
[链接](https://leetcode.cn/problems/lru-cache/description/)

解题思路如下：

（1）首先维护一个双向链表，因为要满足LRU的规则需要频繁地将链表中的元素移动到链表头部，所以选择便于删除的双向链表

（2）还要维护一个key到对应链表结点的映射表，目的是快速地找到在缓存中的key-value对

## LeetCode 0148 排序链表【中等】
[链接](https://leetcode.cn/problems/sort-list/description/)

对于链表而言，似乎更容易使用归并排序算法对其进行排序。

（1）不断地寻找链表中点，将链表一分为二，对左右两部分递归地进行排序

（2）如果递归到只剩一个结点，直接返回即可

（3）对返回的左右两部分有序链表进行合并操作

核心代码如下：

```cpp
ListNode *sort(ListNode *head, ListNode *tail) {
    if (head == nullptr) {
        return head;
    }
    if (head->next == tail) {
        head->next = nullptr;
        return head;
    }

    ListNode *slow = head;
    ListNode *fast = head;
    while (fast != tail) {
        slow = slow->next;
        fast = fast->next;
        if (fast != tail) {
            fast = fast->next;
        }
    }
    ListNode *mid = slow;

    return merge(sort(head, mid), sort(mid, tail));
}

ListNode* sortList(ListNode* head) {
    return sort(head, nullptr);
}
```

## LeetCode 0160 相交链表【简单】
[链接](https://leetcode.cn/problems/intersection-of-two-linked-lists/description/)

解题代码如下所示，思路很简单：让两个指针沿着链表往前走，**一旦到头就将指针值替换成另一条链表的头结点**。这样当两个指针相等时，指针所指向的结点就是两链表相交的结点。

```cpp
ListNode *getIntersectionNode(ListNode *l1, ListNode *l2) {
    ListNode *A = l1;
    ListNode *B = l2;
    while (A != B) {
        A = (A != nullptr) ? A->next : l2;
        B = (B != nullptr) ? B->next : l1;
    }

    return A;
}
```

## LeetCode 0206 反转链表【简单】
[链接](https://leetcode.cn/problems/reverse-linked-list/description/)

反转链表是很多链表题目的基础操作，这里提供两种实现方法。首先是递归法，代码如下所示：

```cpp
ListNode* reverseList(ListNode* head) {
    if (head == nullptr || head->next == nullptr) {
        return head;
    }

    ListNode *newHead = reverseList(head->next);
    head->next->next = head;
    head->next = nullptr;
    return newHead;
}
```

其次是迭代法，代码如下所示：

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode dummy(0, nullptr);
    ListNode *tail;

    while (head != nullptr) {
        tail = head->next;
        head->next = dummy.next;
        dummy.next = head;
        head = tail;
    }

    return dummy.next;
}
```

## LeetCode 0234 回文链表【简单】
[链接](https://leetcode.cn/problems/palindrome-linked-list/description/)

最简单的思路是：

（1）先遍历链表，将序列存在vector中；

（2）根据vector中的内容判断是否为回文链表。

但是该方法的空间复杂度是`O(N)`，如果需要空间复杂度为`O(1)`的方法，可以按照下面的步骤来做：

（1）找到链表的中间结点；

（2）对中间结点后的链表部分进行逆序；

（3）比较链表的前后两半部分是否相同，若是则说明是回文串；否则不是

但是这个方法会比较复杂，代码略。



## LeetCode 0445 两数相加2【中等】
[链接](https://leetcode.cn/problems/add-two-numbers-ii/description/)

解题思路很简单，先把两个链表逆序，然后再模拟加法计算即可。计算得到新结点以头插法的形式插入这样可以避免对结果链表再次进行逆序。


## LeetCode 0705 设计哈希集合【简单】
[链接](https://leetcode.cn/problems/design-hashset/description/)

解题思路很简单，简单写一个拉链法的哈希桶即可，代码略。

## LeetCode 0706 设计哈希映射【简单】
[链接](https://leetcode.cn/problems/design-hashmap/description/)

思路很简单，和LeetCode 0705非常类似，代码略。

