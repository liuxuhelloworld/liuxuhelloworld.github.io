# 题目链接
https://leetcode-cn.com/problems/binary-search/

# 解答过程
二分查找嘛，教科书基础算法，除了计算中间下标时注意下溢出问题，没啥多说的。

```java
	public int search(int[] nums, int target) {
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

		return -1;
	}
```