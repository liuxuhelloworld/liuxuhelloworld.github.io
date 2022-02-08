# 题目链接
https://leetcode-cn.com/problems/search-insert-position/

# 解答过程
这个题目就比较简单了，在二分查找的基础上增加一个插入位置下标的计算而已。稍微想一下两个连续元素的二分查找过程就可以确认最后返回right+1即可

```java
	public int searchInsert(int[] nums, int target) {
		assert nums.length > 0;

		int left = 0, right = nums.length - 1;
		while (left <= right) {
			int mid = left + (right - left) / 2;
			if (nums[mid] == target) {
				return mid;
			} else if (nums[mid] > target) {
				right = mid - 1;
			} else {
				left = mid + 1;
			}
		}

		return right + 1;
	}
```