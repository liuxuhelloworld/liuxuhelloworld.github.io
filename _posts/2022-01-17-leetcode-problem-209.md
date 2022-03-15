# 题目链接
https://leetcode-cn.com/problems/minimum-size-subarray-sum/

# 解答过程
滑动窗口的解法是参考官方题解写的，之前自己写了一个比较难看的双指针解法。滑动窗口的解法理解起来也比较简单，循环不变量是，每次循环先右指针向右扩大窗口，持续扩大直到当前窗口内元素之和大于等于target。然后开始左指针向右缩小窗口，持续缩小直到当前窗口内元素之和小于target。在这个过程中，及时更新满足条件的最小子数组长度。

```java
	public int minSubArrayLen(int target, int[] nums) {
		int min = Integer.MAX_VALUE;
		int left = 0, right = 0;
		int sum = 0;

		while (right < nums.length) {
			sum += nums[right];

			while (sum >= target) {
				min = Math.min(min, right - left + 1);
				sum -= nums[left];
				left++;
			}

			right++;
		}

		return min == Integer.MAX_VALUE ? 0 : min;
	}
```