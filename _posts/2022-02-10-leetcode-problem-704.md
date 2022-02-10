# 题目链接
https://leetcode-cn.com/problems/binary-search/

# 解答过程
额， 这个是真没什么好说的，经典二分查找算法。

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