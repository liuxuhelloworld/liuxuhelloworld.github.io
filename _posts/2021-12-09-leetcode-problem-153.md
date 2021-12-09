# 题目链接
https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/

# 解答过程
思考这个题目的时候，脑子里其实是有和官方题解类似的示意图的，两段错开的上升序列，但是在如何二分舍弃时纠结于同时比较nums[mid]和num[left]、nums[right]的大小关系，而没能抽象出只和nums[right]比较即可的思路。

```java
	public int findMin(int[] nums) {
		int left = 0, right = nums.length - 1;
		while (left < right) {
			int mid = left + (right - left) / 2;
			if (nums[mid] > nums[right]) {
				left = mid + 1;
			} else if (nums[mid] < nums[right]) {
				right = mid;
			}
		}

		return nums[left];
	}
```