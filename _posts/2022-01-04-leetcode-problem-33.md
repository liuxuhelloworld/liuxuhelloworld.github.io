# 题目链接
https://leetcode-cn.com/problems/search-in-rotated-sorted-array/

# 解答过程
这个题目是经典的二分查找的变种，而题目要求又是O(log(N))的时间复杂度，那么直觉上应该还是从二分查找入手。对于二分查找的题目，重点就是二分后如何舍弃一半。经典的二分查找算法中，由于单调递增性，通过比较mid元素和target就可以确认舍弃哪一半。这个题目中，由于rotation破坏了单调递增性，比较mid元素和target不足以确认该舍弃哪一半。关键点就在于单调递增性。那么如何获得单调递增性呢？通过比较mid元素和第一个元素就可以确认\[0..mid\]和\[mid..n-1\]哪一部分是具备单调递增性的，由此再和target比较来确认该舍弃哪一半。

这个题目和Problem4有一定相似之处，都是基于有序数组做二分法的文章，Problem4是把单个数组的有序性拆分为两个有序数组，Problem33是通过rotation把单个数组的有序性变成两段有序子数组，问题的关键点都是基于破坏后的有序性做二分法。

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

相似题目
- [Problem4-Median of Two Sorted Arrays](2022-11-07-leetcode-problem-4.md)
- [Problem81-Search in Rotated Sorted Array](2023-11-09-leetcode-problem-81.md)
- [Problem153-Find Minimum in Rotated Sorted Array](2021-12-09-leetcode-problem-153.md)