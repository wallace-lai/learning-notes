# 堆

作者：wallace-lai </br>
发布：2020-09-01 </br>
更新：2020-09-01 <br>

## LeetCode 0023 合并 K 个升序链表

给你一个链表数组，每个链表都已经按升序排列。请你将所有链表合并到一个升序链表中，返回合并后的链表。

```
    输入：lists = [[1,4,5],[1,3,4],[2,6]]
    输出：[1,1,2,3,4,4,5,6]
    解释：链表数组如下：
        [
        1->4->5,
        1->3->4,
        2->6
        ]
    将它们合并到一个有序链表中得到。
        1->1->2->3->4->4->5->6
```

使用小顶堆合并链表的思路：定义一个存储链表节点的小顶堆，先让三个链表的头节点入堆，随后不断地重复以下过程——将值最小的链表头出堆，将其并入合并后链表的末尾，若该表头的next指针不为空则将其next入堆。直到整个堆中元素全部被移出为止。

我们先把存储 `*ListNode`的小顶堆用Go语言实现一下。

```go
type Heap []*ListNode

func (h Heap) Len() int {
    return len(h)
}

func (h Heap) Less(i, j int) bool {
    return h[i].Val < h[j].Val
}

func (h Heap) Swap(i, j int) {
    tmp := h[i]
    h[i] = h[j]
    h[j] = tmp
}

func (h *Heap) Push(x any) {
    *h = append(*h, x.(*ListNode))
}

func (h *Heap) Pop() any {
    old := *h
    n := len(old)
    x := old[n - 1]
    *h = old[0 : n - 1]
    return x
}
```

随后在此基础上进行链表的合并，先将所有的表头节点加入到堆中，然后不断地从堆中弹出值最小的节点将其加入到合并后的链表末尾。若最小节点的next不为nil则将next加入堆中。

```go
func mergeKLists(lists []*ListNode) *ListNode {
    n := len(lists)
    if n == 0 {
        return nil
    }

    h := &Heap{}
    heap.Init(h)
    for i := 0; i < n; i++ {
        if lists[i] != nil {
            heap.Push(h, lists[i])
        }
    }

    dummy := ListNode {0, nil}
    tail := &dummy
    for h.Len() > 0 {
        node := heap.Pop(h).(*ListNode)
        tail.Next = node
        tail = tail.Next
        if node.Next != nil {
            heap.Push(h, node.Next)
        }
    }

    return dummy.Next
}
```


## LeetCode 0313 超级丑数

## LeetCode 0355 设计推特

## LeetCode 0373 查找和最小的K对数字

## LeetCode 0378 有序矩阵中第k小的元素

