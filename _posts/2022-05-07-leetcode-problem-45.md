# 题目链接
https://leetcode-cn.com/problems/jump-game-ii/

# 解答过程
这个题目是挺久以前写的了，看了下和官方题解并不相似，说明当时不是参考官方题解写的，但是真的一点印象也没有了......

这个题目如果从递归的思路来看，还是有比较清晰的解题思路的，可以倒序遍历数组，对于能一步跳到结尾元素的X，问题变为从开头元素跳到X的最小跳数，而这恰好是原问题的更小规模的子问题，由此递归直到开头元素。

一个最优解问题，能通过递归解决，递归过程中有可能重复处理子问题，这些都提示应当使用动态规划。动态规划的关键是定义合适的状态转移方程，这个题目里我们可以定义dp[i]表示从i元素到结尾元素的最小跳数，与上述递归思路类似，同样是倒叙遍历数组，dp[i]的值可以从dp[i+1]到dp[len-1]的值计算而得。

```java
	public int jump(int[] nums) {
		int len = nums.length;

		if (len == 0 || len == 1) {
			return 0;
		}

		int[] jumps = new int[len]; // jumps[i] means the minimum jumps from nums[i] to the last element
		for (int i = 0; i < len-1; i++) {
			jumps[i] = -1;
		}
		jumps[len-1] = 0;

		for (int i = len-2; i >= 0; i--) {
			jumps[i] = minJumps(jumps, i, nums[i]);
		}

		return jumps[0];
	}

	private int minJumps(int[] jumps, int i, int val) {
		if (val == 0) {
			return -1;
		}

		int min = Integer.MAX_VALUE;
		for (int j = i + 1; j <= i + val && j < jumps.length; j++) {
			if (jumps[j] == -1) {
				continue;
			}
			min = Math.min(min, 1 + jumps[j]);
		}

		if (min == Integer.MAX_VALUE) {
			return -1;
		}

		return min;
	}
```

官方题解的贪心算法也有点意思，有时间再写下贪心算法的代码。