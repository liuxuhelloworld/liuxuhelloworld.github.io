# 题目链接
https://leetcode-cn.com/problems/subarray-product-less-than-k/

# 解答过程
对于这个题目，我一开始想到的思路是，对于任意一个nums[i]，无论是以它为起点的子数组或者以它为终点的子数组，满足乘积小于K的，数目是确定的，那么对每个nums[i]计算这个数目，再累加起来就可以了。但是如何计算以某个nums[i]开始的满足乘积小于K的子数组的个数呢？我想到的是按子数组长度从小到大挨个乘起来，每累加乘一个，判断一次与K的大小关系。然后直觉上就觉得这样太笨了，看了一下官方题解，官方题解双指针的解法我是觉得代码不大好理解，但是有一点倒是提醒了我，就是并不需要按子数组长度从小到大挨个乘起来，只需要找到以某个num[i]开始的满足乘积小于K的最大的那个子数组就可以了，假设这个最大子数组的结尾下标是j，那么对应的以nums[i]开始的满足乘积小于K的子数组的个数就是j-i+1.

```java
	public int numSubarrayProductLessThanK(int[] nums, int k) {
		int cnt = 0;

		for (int start = 0; start < nums.length; start++) {
			int end = endIndexOfMaxSubarrayProductLessThanKStartFrom(nums, start, k);
			if (end >= start) {
				cnt += end - start + 1;
			}
		}

		return cnt;
	}

	private int endIndexOfMaxSubarrayProductLessThanKStartFrom(int[] nums, int start, int k) {
		int product = 1;
		int i = start;
		for (; i < nums.length; i++) {
			product *= nums[i];
			if (product >= k) {
				break;
			}
		}

		return i - 1;
	}
```

提交以后是没有问题的，只是效率比较差，但我觉得这个代码比官方题解的好理解。