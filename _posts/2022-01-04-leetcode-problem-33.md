# 题目链接
https://leetcode-cn.com/problems/search-in-rotated-sorted-array/

# 解答过程
看了下提交记录，这个题目是差不多一年前写的了，首次提交用递归写的，额，一点印象都没有了。后面看了官方题解，确实简洁很多，也更能体现二分查找的特点。对于二分查找的题目，重点就是二分后如何舍弃一半。

```java
  public int search(int[] nums, int target) {
    if (nums.length == 0) {
      return -1;
    }

    int left = 0, right = nums.length - 1;

    while (left <= right) {
      int mid = left + (right - left)/2;
      if (nums[mid] == target) {
        return mid;
      }

      if (nums[0] <= nums[mid]) {
        // left part is ordered
        if (target >= nums[0] && target < nums[mid]) {
          right = mid - 1;
        } else {
          left = mid + 1;
        }
      } else {
        // right part is ordered
        if (target > nums[mid] && target <= nums[nums.length-1]) {
          left = mid + 1;
        } else {
          right = mid - 1;
        }
      }
    }

    return -1;
  }
```