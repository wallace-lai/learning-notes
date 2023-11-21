# 动态规划

作者：wallace-lai
时间：2022-06-07

## 算法原理
动态规划的三个核心要素：
- 状态转移方程
- 最优子结构
- 重叠子问题


## LeetCode 0005
## LeetCode 0063
## LeetCode 0064
## LeetCode 0070

设爬`n`阶所需的方法数为`f(n)`，则可得状态转移方程：

$$
f(n) = f(n - 1) + f(n - 2)
$$

```go
func climbStairs(n int) int {
	f := []int {1, 2, 0}

	if n < 3 {
		return f[n - 1]
	}
	for i := 3; i <= n; i++ {
		f[2] = f[0] + f[1]
		f[0] = f[1]
		f[1] = f[2]
	}
	return f[2]
}
```

## LeetCode 0072
## LeetCode 0097
## LeetCode 0120
## LeetCode 0123
## LeetCode 0139
## LeetCode 0188
## LeetCode 0198
设`$f(i)$`表示前`$i$`间房屋所能偷窃到的最高金额，对于第`$i$`间房屋只有两种选择，即：

（1）偷窃第`$i - 1$`间房屋，不偷窃第`$i$`间屋子，此时的最高金额为`$f(i - 1)$`；

（2）不偷窃第`$i - 1$`间房屋，偷窃第`$i$`间房屋，此时的最高金额为`$f(i - 2) + nums[i]$`；

二者取最大值就是`$f(i)$`的值，于是得到如下的状态转移方程：

$$
f(i) = max(f(i - 1), f(i - 2) + nums[i])
$$

边界条件为：

（1）只有一间屋子时，偷窃之。

$$
f(0) = nums[0]
$$

（2）有两间屋子时，选择其中价值最高的偷窃之。

$$
f(1) = max(nums[0], nums[1])
$$

最终答案为`$f(n - 1)$`，其中`$n$`为房屋总数。

```go
func getMax(a, b int) int {
	if a >= b {
		return a
	}

	return b
}

func rob(nums []int) int {
	n := len(nums)
	if n == 1 {
		return nums[0]
	}

	f0 := nums[0]
	f1 := getMax(nums[0], nums[1])
	f2 := 0

	if n == 2 {
		return f1
	}
	for i := 2; i < n; i++ {
		f2 = getMax(f1, f0 + nums[i])
		f0 = f1
		f1 = f2
	}
	return f2
}
```


## LeetCode 0221
## LeetCode 0300
## LeetCode 0322