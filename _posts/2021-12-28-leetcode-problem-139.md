# 题目链接
https://leetcode-cn.com/problems/word-break/

# 解答过程
这个题目是挺久以前写的了，第一时间也没有思路，看了下提交记录，发现还写过两种解法，但是一点印象也没有，很尴尬......


一开始写了个深度优先遍历的版本，理论上应该是对的，但是执行超时。代码如下：
```java
    public boolean wordBreak(String s, List<String> wordDict) {
		return dfs(s, -1, 0, wordDict);
	}

	private boolean dfs(String s, int segmented, int cur, List<String> wordDict) {
		if (segmented == s.length() - 1) {
			return true;
		}

		if (cur == s.length()) {
			return false;
		}

		String str = s.substring(segmented + 1, cur + 1);
		if (wordDict.contains(str)) {
			if (dfs(s, cur, cur + 1, wordDict)) {
				return true;
			}
		}

		if (dfs(s, segmented, cur + 1, wordDict)) {
			return true;
		}

		return false;
	}
```

其实想一想，这个题目算是比较典型的动态规划题目了，想清楚状态转移方程即可。我们定义dp[i]表示s[0..i]对应的子串是否满足wordBreak，那么怎么得到dp[i+1]呢？
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