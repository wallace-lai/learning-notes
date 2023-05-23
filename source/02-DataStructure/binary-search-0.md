# 二分查找

作者：wallace-lai </br>
时间：2023-05-24 </br>
更新：2023-06-03 </br>

## 理解算法模板

二分查找算法原理很简单，但实现起来相当地tricky，里面有无数的细节在等着坑你。参考labuladong的算法小抄，先把最常见的三种模板的代码实现写出来，然后再一一拆解为什么要这样实现。

### 经典二分查找

经典二分查找要做的就是在有序数组`nums`中查找给定的`target`在数组中的下标，如果不存在则返回`-1`。

```go
func binarySearch(nums []int, target int) int {
    n := len(nums)
    if n == 0 {
        return -1
    }

    left := 0
    right := n - 1
    for left <= right {
        mid := left + (right - left) / 2
        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            left = mid + 1
        } else if nums[mid] > target {
            right = mid - 1
        }
    }

    return -1
}
```

对于经典的二分查找算法，由于数组中不涉及多个相同`target`值，所以一旦搜索到了`target`立即返回其下标即可。否则不断地收缩左右边界进入下一轮循环即可。

对于上面的代码，有以下几点注意的地方：

- 所有代码的默认搜索区间都是左右闭合的
- 循环条件是`left <= right`
- 循环结束时有`left == right + 1`成立


### 寻找左侧边界
如果数组中有多个相同的`target`值，要求你返回对应`target`的最左侧边界下标，这时要怎么做呢？我们先把搜索框架写出来，当`nums[mid]`和`target`相等时，我们选择继续收缩右边界。

```go
func leftBound(nums []int, target int) int {
	n := len(nums)
	if n == 0 {
		return -1
	}

	left := 0
	right := n - 1
	for left <= right {
		mid := left + (right - left) / 2
		if nums[mid] == target {
			right = mid - 1	// 收缩右侧边界
		} else if nums[mid] < target {
			left = mid + 1	// 收缩左侧边界
		} else if nums[mid] > target {
			right = mid - 1	// 收缩右侧边界
		}
	}

    // ?
}
```

当不满足循环条件之后，我们可以分以下几种情况讨论如何返回正确的值。

- （1）target不存在于数组中
  - 1.1 target比数组中的数都要大
  - 1.2 target比数组中的数都要小
  - 1.3 target在数组最大最小数范围内
  
- （2）target存在于数组中

对于情况1.1，由于`nums[mid] < target`在整个搜索过程中都是存在的，因此`left`会不断地增大。循环结束后`left`和`right`的值为`left == n && right == n - 1`。因此对于情况1.1，应该返回`-1`。

```go
if left == n {
    return -1
}
```

对于情况1.2，与情况1.1类似，最终的结果是`left == 0 && right == -1`。此时，同样应该返回`-1`。

```go
// v1 : wrong
if right == -1 {
    return -1
}

// v2
if nums[left] != target {
    return -1
}
```
注意不能使用`v1`的方法用于判定情况1.2，这是错误的。比如数组为`[4, 4, 4, 5, 6]`，target为`4`，循环结束时`right`值也是`-1`，但显然你不能返回`-1`。

对于情况1.3，最终的结果是`left`指针停留在数组中第一个大于`target`值的下标处，此时，应该返回`-1`。

```go
if nums[left] != target {
    return -1
}
```

对于情况（2），循环结束时，`left`指针所在的就是我们要求的左侧边界的位置，此时应该返回`left`。

综上，我们可以得到完整的寻找左侧边界代码如下。

```go
func leftBound(nums []int, target int) int {
	n := len(nums)
	if n == 0 {
		return -1
	}

	left := 0
	right := n - 1
	for left <= right {
		mid := left + (right - left) / 2
		if nums[mid] == target {
			right = mid - 1	// 收缩右侧边界
		} else if nums[mid] < target {
			left = mid + 1	// 收缩左侧边界
		} else if nums[mid] > target {
			right = mid - 1	// 收缩右侧边界
		}
	}

    // 情况1.1
    if left == n {
        return -1
    }
    // 情况1.2与1.3
    if nums[left] != target {
        return -1
    }

    // 情况2
    return left
}
```


### 寻找右侧边界
寻找右侧边界与寻找左侧边界类似，代码模板如下。
```go
func rightBound(nums []int, target int) int {
	n := len(nums)
	if n == 0 {
		return -1
	}

	// 搜索区间[left, right]
	left := 0
	right := n - 1

	for left <= right {
		mid := left + (right - left) / 2
		if nums[mid] == target {
			left = mid + 1	// 收缩左侧边界
		} else if nums[mid] < target {
			left = mid + 1	// 收缩左侧边界
		} else if nums[mid] > target {
			right = mid - 1	// 收缩右侧边界
		}
	}

	// 情况1.2
	if right == -1 {
		return -1
	}

	// 情况1.1和1.3
	if nums[right] != target {
		return -1
	}

	// 情况2
	return right
}
```

## LeetCode 0035
题目的大意是在 **无重复元素** 的 **升序** 数组中查找target的下标，如果target不存在则返回target会被顺序插入的位置。显然这题是经典二分算法的变形，参考模板可以写出如下的AC代码。

```go
func searchInsert(nums []int, target int) int {
    left := 0
    right := len(nums) - 1

    for left <= right {
        mid := left + (right - left) / 2
        if (nums[mid] == target) {
            return mid
        } else if (nums[mid] < target) {
            left = mid + 1
        } else if (nums[mid] > target) {
            right = mid - 1
        }
    }

    return left
}
```

对于以上代码，有几个需要注意的点：
（1）搜索区间是左右闭合的，所以循环条件是`left <= right` ，循环结束时满足 `left == right + 1`
（2）由于当下标`mid`处的值小于`target`时，`left`指针总是往前加1。所以当`target`不存在时，循环结束时`left`前面全是小于`target`的值，因此`left`就是`target`应该顺序插入的位置

##  LeetCode 0074
题目大意是在一个按列和按行升序的二维数组中搜索是否存在target，存在返true，否则返false。这题也是经典二分算法的变形，不过有几点需要注意的地方。

```go
func searchMatrix(matrix [][]int, target int) bool {
    m := len(matrix)
    n := len(matrix[0])

    // 先在第1列中进行二分查找
    left := 0
    right := m - 1
    for left <= right {
        mid := left + (right - left) / 2
        if matrix[mid][0] == target {
            return true
        } else if matrix[mid][0] < target {
            left = mid + 1
        } else if matrix[mid][0] > target {
            right = mid - 1
        }
    }

    // 若target比matrix[0][0]还小
    if right == -1 {
        return false
    }

    // 再在第row行进行二分查找
    row := right
    left = 0
    right = n - 1
    for left <= right {
        mid := left + (right - left) / 2
        if matrix[row][mid] == target {
            return true
        } else if matrix[row][mid] < target {
            left = mid + 1
        } else if matrix[row][mid] > target {
            right = mid - 1
        }
    }

    return false
}
```

## 参考资料
1. labuladong的算法小抄