# 题目链接
https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/

# 解答过程
这个题目属于二分查找的变种，直觉上就是连续二分查找。看了官方题解，我觉得还不如这种处理容易理解。另外，相比Problem4那一组二分查找的变种，这一组简单很多。

```java
  public int[] searchRange(int[] nums, int target) {
    if (nums.length == 0) {
      return new int[] {-1, -1};
    }

    int leftest = -1, rightest = -1;
    int index = binarySearch(nums, 0, nums.length-1, target);
    if (index == -1) {
      return new int[] {-1, -1};
    } else {
      leftest = rightest = index;
    }

    while ((index = binarySearch(nums, 0, leftest-1, target)) != -1) {
      leftest = index;
    }

    while ((index = binarySearch(nums, rightest+1, nums.length-1, target)) != -1) {
      rightest = index;
    }

    return new int[] {leftest, rightest};
  }

  private int binarySearch(int[] nums, int left, int right, int target) {
    if (left > right) {
      return -1;
    }

    if ((left >= 0 && target < nums[left])
        || (right <= nums.length-1 && target > nums[right])) {
      return -1;
    }

    while (left <= right) {
      int mid = left + (right - left)/2;
      if (nums[mid] == target) {
        return mid;
      } else if (nums[mid] > target) {
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    }

    return -1;
  }
```

# 相似题目
- [Problem278-First Bad Version](2022-02-10-leetcode-problem-278.md)