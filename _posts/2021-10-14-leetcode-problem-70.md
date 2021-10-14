# 题目链接
https://leetcode-cn.com/problems/climbing-stairs/

# 解答过程
从动态规划的角度去考虑，还是很容易想到怎么解决这个问题的。从第1层到第i层，每次只能走1层或2层，所以无论怎么走，最终到第i层的可能走法就是到第i-2层的可能走法加上到第i-1层的可能走法之和。也就是说，就是fibonacci数列。

```java
	public int climbStairs(int n) {
		if (n == 1) {
			return 1;
		}

		if (n == 2) {
			return 2;
		}

		int[] dp = new int[n + 1];
		dp[1] = 1;
		dp[2] = 2;

		for (int i = 3; i <= n; i++) {
			dp[i] = dp[i - 1] + dp[i - 2];
		}

		return dp[n];
	}
```