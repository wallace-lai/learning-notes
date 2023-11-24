# 【Go标准库】堆

作者：wallace-lai <br>
时间：2023-05-27 <br>
更新：2023-11-24 <br>

## 一、堆的实现

### 1.1 接口定义
Go语言对泛型的支持还不完善，但有意思的是Go标准库利用接口实现了堆这种数据结构。我们先看标准库中的堆是如何实现的，最后再介绍如何使用它。

```go
import "container/heap"

type Interface interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}

type Interface interface {
    sort.Interface
    Push(x any) // add x as element Len()
    Pop() any   // remove and return element Len() - 1.
}
```

从接口类型的定义中可以看到，要想使用heap中的算法我们必须要实现接口中定义的这五个方法。尤其要注意的是Push方法要求用户将新插入元素放在数组的末尾处，而Pop要求用户从数组中删除末尾元素并返回该末尾元素。

### 1.2 建堆
我们都知道堆有两种调整方式，分别是自顶向下的down和自底向上的up。这两种操作的实现方式如下所示。

```go
func up(h Interface, j int) {
    for {
        i := (j - 1) / 2 // parent
        if i == j || !h.Less(j, i) {
            break
        }
        h.Swap(i, j)
        j = i
    }
}
```

up操作一般在堆中添加元素后调整堆结构时调用。j是插入元素在堆中的下标，我们需要和其对应的父节点i的值进行比较，如果j比i小（默认是小顶堆）那么就需要将j和i的值交换。然后是比较交换后的父节点与父节点的父节点，如此重复这个过程，直到遇到根节点`i == j` 或者当前节点不比父节点小`!h.Less(j, i)`为止。

遇到根节点了意味着整个调整过程都结束了，如果是当前节点不比父节点小则意味着整棵二叉树已经符合了小顶堆的性质，可以提前退出无需再调整了。

```go
func down(h Interface, i0, n int) bool {
    i := i0
    for {
        j1 := 2*i + 1
        if j1 >= n || j1 < 0 { // j1 < 0 after int overflow
            break
        }
        j := j1 // left child
        if j2 := j1 + 1; j2 < n && h.Less(j2, j1) {
            j = j2 // = 2*i + 2  // right child
        }
        if !h.Less(j, i) {
            break
        }
        h.Swap(i, j)
        i = j
    }
    return i > i0
}
```

down操作一般是删除了堆顶元素之后调整堆结构时调用。对于下标i，我们先计算它的左右子树的下标j1和j2，取它们中最小的那个赋值给j。如果i比j小，那么没有继续调整的必要，break退出循环，否则交换i和j对应的值，同时将j赋值给i继续循环下去直到到达堆的最底层为止。如果在循环中发生了交换`i = j`，那么i的值一定会大于i0，所以down的返回值为true表示的是down的过程中发生了交换（调整）。

```go
func Init(h Interface) {
    // heapify
    n := h.Len()
    for i := n/2 - 1; i >= 0; i-- {
        down(h, i, n)
    }
}
```

实现了down和up操作，那么堆最核心的问题也就解决了。对于建堆过程，我们只需要从下标`n / 2 - 1`处不断地自顶向下调整即可，一直调整到根节点（i == 0）为止即可完成建堆。

### 1.3 插入

```go
func Push(h Interface, x any) {
    h.Push(x)
    up(h, h.Len()-1)
}
```

插入操作比较简单，首先调用用户提供的Push方法将元素存储到数组的末尾，然后从新插入元素的下标开始自底向上调整堆的结构。直到整棵二叉树满足了最小堆的性质为止。

看到这里一切都基本了然了，标准库heap要求用户使用数组来存储元素以满足当前节点i对应的左右子树节点是`2 * i + 1` 和`2 * i + 2`的这层关系。这与我们平时手写堆的过程是一样的。

### 1.4 删除根节点

```go
func Pop(h Interface) any {
    n := h.Len() - 1
    h.Swap(0, n)
    down(h, 0, n)
    return h.Pop()
}
```

删除根节点意味着弹出的是数组中最小值（默认为小顶堆），首先调用Swap将根节点的值和数组末尾的值交换。然后调用down从根节点处自顶向下调整，注意传入的数组长度n是Len减去1，意味着此时调整范围不包括最末尾元素。最后调用用户提供的Pop方法将末尾元素返回即可

### 1.5 删除任意节点

```go
func Remove(h Interface, i int) any {
    n := h.Len() - 1
    if n != i {
        h.Swap(i, n)
        if !down(h, i, n) {
            up(h, i)
        }
    }
    return h.Pop()
}
```

删除任意节点比较复杂，而且实际上在堆中基本不会用到这个方法。如果待删除的节点i刚好是数组末尾节点，那么无需任何操作直接调用用户提供的Pop将末尾元素移除并返回即可，否则我们需要从节点i开始往下调整堆。**这里为啥要调用up** ？

### 1.6 修改节点值后调整堆

```go
func Fix(h Interface, i int) {
    if !down(h, i, h.Len()) {
        up(h, i)
    }
}
```

当下标i处的值被修改后，需要调用Fix去重新调整堆结构以满足小顶堆的性质，原理与Remove类似。**这里为啥要调用up**？

## 二、标准库的使用
在分析完了heap的实现之后，对于如何使用heap应该非常了解了。我们尝试做几个heap的练习题，更多的习题见算法与数据结构部分的内容。

### 2.1 LeetCode 0215
```
给定整数数组 nums 和整数 k，请返回数组中第 k 个最大的元素。
请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。
你必须设计并实现时间复杂度为 O(n) 的算法解决此问题。

示例 1:

    | 输入: [3,2,1,5,6,4], k = 2
    | 输出: 5

示例 2:

    | 输入: [3,2,3,1,2,4,5,5,6], k = 4
    | 输出: 4
```
思路：我们可以将整个数组建立成一个大顶堆，然后不断地弹出二叉堆的根节点，第k个弹出的根节点就是第k个最大的元素。

```go
type MaxHeap []int

func (h MaxHeap) Len() int {
    return len(h)
}

func (h MaxHeap) Less(i, j int) bool {
    return h[i] > h[j]
}

func (h MaxHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}

func (h *MaxHeap) Push(x any) {
    *h = append(*h, x.(int))
}

func (h *MaxHeap) Pop() any {
    old := *h
    n := len(old)
    x := old[n - 1]
    *h = old[0 : n - 1]
    return x
}
```

我们先将大顶堆需要的五个接口给实现了，然后在此基础上求第k个最大的元素

```go
func findKthLargest(nums []int, k int) int {
    h := MaxHeap{}
    heap.Init(&h)

    for i := 0; i < len(nums); i++ {
        heap.Push(&h, nums[i])
    }

    result := 0
    for i := 0; i < k; i++ {
        result = heap.Pop(&h).(int)
    }

    return result
}
```

注意：使用大顶堆的算法时间复杂度是`O(NlogN)`，没有达到题目的要求。如果要`O(N)`的复杂度，则需要使用快速选择算法。


### 2.2 LeetCode 0373
```
给定两个以 升序排列 的整数数组 nums1 和 nums2 , 以及一个整数 k 。
定义一对值 (u,v)，其中第一个元素来自 nums1，第二个元素来自 nums2 。
请找到和最小的 k 个数对 (u1,v1),  (u2,v2)  ...  (uk,vk) 。

示例 1:

    | 输入: nums1 = [1,7,11], nums2 = [2,4,6], k = 3
    | 输出: [1,2],[1,4],[1,6]
    | 解释: 返回序列中的前 3 对数：
        | [1,2],[1,4],[1,6],[7,2],[7,4],[11,2],[7,6],[11,4],[11,6]

示例 2:

    | 输入: nums1 = [1,1,2], nums2 = [1,2,3], k = 2
    | 输出: [1,1],[1,1]
    | 解释: 返回序列中的前 2 对数：
    | [1,1],[1,1],[1,2],[2,1],[1,2],[2,2],[1,3],[1,3],[2,3]

示例 3:

    | 输入: nums1 = [1,2], nums2 = [3], k = 3 
    | 输出: [1,3],[2,3]
    | 解释: 也可能序列中所有的数对都被返回:[1,3],[2,3]
```

思路：

我们将二元组两元素在两个数组中的下标作为小顶堆中的元素，假设当前最小的`n`对二元组为

$$
(a_1, b_1), ..., (a_n, b_n)
$$

那么第`n + 1`小的二元组的下标选择范围为

$$
(a_1 + 1, b_1), (a_1, b_1 + 1) ..., (a_n + 1, b_n), (a_n, b_n + 1)
$$

于是，我们首先将第一小的二元组的下标`(0, 0)`压入小顶堆中，每次从小顶堆中取出一个二元组后扩展下一个二元组可能的值并压入小顶堆中，直到取出了前k个为止。

注意：
为了避免重复，我们先将`nums1`的前k个索引加入到小顶堆中，每次从队列中取出元素对时，只需要将`nums2`的索引增加即可。这样可以避免重复加入元素的问题

```go
var array1 []int
var array2 []int

type Item struct {i, j int}
type MinHeap []Item

func (h MinHeap) Len() int {
    return len(h)
}

func (h MinHeap) Less(i, j int) bool {
    x1, y1 := h[i].i, h[i].j
    x2, y2 := h[j].i, h[j].j
    return array1[x1] + array2[y1] < array1[x2] + array2[y2]
}

func (h MinHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}

func (h *MinHeap) Push(x any) {
    *h = append(*h, x.(Item))
}

func (h *MinHeap) Pop() any {
    old := *h
    n := len(old)
    x := old[n - 1]
    *h = old[0 : n - 1]
    return x
}
```

为了减少小顶堆实现的复杂性，使用了两个全局变量`array1`和`array2`。

```go
func kSmallestPairs(nums1 []int, nums2 []int, k int) [][]int {
    array1 = nums1
    array2 = nums2

    m := len(nums1)
    n := len(nums2)

    h := MinHeap{}
    heap.Init(&h)
    for i := 0; i < m; i++ {
        heap.Push(&h, Item{i, 0})
    }

    result := [][]int{}
    for h.Len() > 0 && len(result) < k {
        item := heap.Pop(&h).(Item)
        i, j := item.i, item.j
        result = append(result, []int{nums1[i], nums2[j]})
        if j + 1 < n {
            heap.Push(&h, Item{i, j + 1})
        }
    }

    return result
}
```
