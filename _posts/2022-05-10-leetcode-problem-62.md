# 题目链接
https://leetcode.cn/problems/unique-paths/

# 解答过程
这个题目是挺久以前写的了，看了下提交记录，一开始还写了个回溯法的解法，从题目的形式来看，确实有很多回溯法的题目都是遍历二维数组，当时可能是用回溯法硬套的这个题目。其实回溯法的逻辑应该是正确的，但是会超时，递归的层级太多了。反过来说，一旦能想到用动态规划来解决这个题目，会有豁然开朗的感觉，算是比较典型的动态规划题目了。状态转移方程也很符合直觉，定义dp\[i\]\[j]表示从[0, 0]到[i, j]的路径数，dp\[i\]\[j\]的计算只依赖于dp\[i-1\]\[j\]和dp\[i\]\[j-1\]

```java
	public int uniquePaths(int m, int n) {
		int[][] dp = new int[m][n]; // dp[i][j] represents the unique paths from [0,0] to [i,j]
		for (int i = 0; i < m; i++) {
			dp[i][0] = 1;
		}
		for (int j = 0; j < n; j++) {
			dp[0][j] = 1;
		}

		for (int i = 1, j = 1; i < m && j < n; i++, j++) {
			for (int k = j; k < n; k++) {
				dp[i][k] = dp[i-1][k] + dp[i][k-1];
			}
			for (int k = i; k < m; k++) {
				dp[k][j] = dp[k-1][j] + dp[k][j-1];
			}
		}

		return dp[m-1][n-1];
	}
```