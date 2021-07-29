# 题目链接
https://leetcode-cn.com/problems/squares-of-a-sorted-array/

# 解答过程
额，这个题吧，看了官方题解才写的，之前也想到了归并排序，但总纠结是不是有更简洁的方式。从首尾向中间做归并，这个确实没想到。
```java
	public int[] sortedSquares(int[] nums) {
		assert nums.length > 0;

		int len = nums.length;

		int[] squares = new int[len];

		int i = 0, j = len - 1, k = len - 1;
		while (i <= j) {
			if (Math.abs(nums[i]) >= Math.abs(nums[j])) {
				squares[k--] = nums[i] * nums[i];
				i++;
			} else {
				squares[k--] = nums[j] * nums[j];
				j--;
			}
		}

		return squares;
	}
```