# 数组

作者：wallace-lai </br>
发布：2024-02-25 </br>
更新：2024-08-09 <br>

## 0. 其他

## 1. 前缀和

### LeetCode 0303
### LeetCode 0304
### LeetCode 0724
### LeetCode 1314
### 剑指Offer 0013

## 2. 前缀和加哈希表

### LeetCode 0325
### LeetCode 0437
### LeetCode 0523
### LeetCode 0525
### LeetCode 0560
### LeetCode 0713
### LeetCode 1124
### LeetCode 1658
### 剑指Offer 0010
### 剑指Offer 0011


## 3. 前缀积

### LeetCode 0238

## 4. 双指针

### LeetCode 0026

思路：

（1）用slow指针指向当前数组前半部分没有重复项的最后一个元素，用fast指针指向当前需要判断的元素；

（2）如果slow和fast指针所指向的元素不相同，说明遇到了不同元素，将fast所指向元素加入到没有重复项的子数组的后面；如果相同，则fast指针自增，然后啥也不做；

（3）重复步骤（2），直到fast指针到达末尾；

```cpp
    int removeDuplicates(vector<int>& nums) {
        if (nums.size() == 0) {
            return 0;
        }

        int slow = 0;
        int fast = 0;
        while (fast < nums.size()) {
            if (nums[fast] != nums[slow]) {
                slow++;
                nums[slow] = nums[fast];
            }
            fast++;
        }

        return slow + 1;
    }
```

类似题型：

（1）LeetCode 0083