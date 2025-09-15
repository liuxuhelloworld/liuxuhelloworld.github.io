# 题目链接
https://leetcode-cn.com/problems/search-insert-position/

# 解答过程
二分查找的变种，只要想清楚在target不存在时，最后跳出循环后left和right的关系，以及target应该放哪就可以了。最终一定是[right,left]这样一个关系结束循环，而right+1正好是target应当处于的位置。
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