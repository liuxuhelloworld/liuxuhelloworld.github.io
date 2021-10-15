# 题目链接
https://leetcode-cn.com/problems/house-robber/

# 解答过程
这个题目从动态规划的角度考虑，并不难想到如何解决。我们定义一个dp数组，dp[i]表示num前i个元素对应的最大抢劫金钱数。那么如何从较小规模的dp元素值得到dp[i]的值呢？如果dp[i]对应的结果包含num第i个元素，那么最多抢劫到num[i-1]+dp[i-2]的钱；如果dp[i]对应的结果不包含num第i个元素，那么最多抢劫到dp[i-1]的钱。所以，dp[i]的值就是这两者中的较大者。

```java
	public int rob(int[] nums) {
		int len = nums.length;

		int[] dp = new int[len+1];
		dp[0] = 0;
		dp[1] = nums[0];

		for (int i = 2; i <= len; i++) {
			int robWithI = nums[i-1] + dp[i-2];
			int robWithoutI = dp[i-1];
			dp[i] = Math.max(robWithI, robWithoutI);
		}

		return dp[len];
	}
```