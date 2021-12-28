# 题目链接
https://leetcode-cn.com/problems/word-break/

# 解答过程
这个题目我是按memoization标签搜出来的，一开始写了个类似回溯法的递归版本，结果就是超时。其实想一想，这个题目算是比较典型的动态规划题目了，想清楚状态转移方程即可。我们定义dp[i]表示s[0..i]对应的子串是否满足wordBreak，那么怎么得到dp[i+1]呢？
- 如果dp[i]是true并且wordDict包含s[i, i+1]，那么dp[i+1]为true，否则继续
- 如果dp[i-1]是true并且wordDict包含s[i-1, i+1]，那么dp[i+1]为true，否则继续
- 如果dp[i-2]是true并且wordDict包含s[i-2, i+1]，那么dp[i+1]为true，否则继续
- ......
- 如果dp[0]是true并且wordDict包含s[0, i+1]，那么dp[i+1]为true

```java
	public boolean wordBreak(String s, List<String> wordDict) {
		int length = s.length();

		boolean[] dp = new boolean[length + 1]; // dp[i] represents whether s[0..i-1] fits wordBreak
		dp[0] = true;

		for (int i = 1; i <= length; i++) {
			for (int j = i - 1; j >= 0; j--) {
				if (dp[j] && wordDict.contains(s.substring(j, i))) {
					dp[i] = true;
					break;
				}
			}
		}

		return dp[length];
	}
```

