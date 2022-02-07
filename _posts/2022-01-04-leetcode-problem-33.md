# 题目链接
https://leetcode-cn.com/problems/search-in-rotated-sorted-array/

# 解答过程
看了下提交记录，这个题目是差不多一年前写的了，首次提交用递归写的，额，一点印象都没有了。后面看了官方题解，确实简洁很多，也更能体现二分查找的特点。对于二分查找的题目，重点就是二分后如何舍弃一半。经典的二分查找算法中，由于单调递增性，通过比较mid元素和target就可以确认舍弃哪一半。这个题目中，由于rotation破坏了单调递增性，比较mid元素和target不足以确认该舍弃哪一半。关键点就在于单调递增性。如何获取单调递增性呢？通过比较mid元素和第一个元素就可以确认\[0..mid\]和\[mid..n-1\]哪一部分是具备单调递增性的，由此再和target比较来确认该舍弃哪一半。

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