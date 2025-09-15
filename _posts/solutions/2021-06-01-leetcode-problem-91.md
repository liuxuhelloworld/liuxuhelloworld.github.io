# 题目链接
https://leetcode-cn.com/problems/decode-ways/

# 解答过程
对于这个题目，很自然得就想到用动态规划解决，dp[i]表示长度为i的子串的解码方式数量，dp[0]初始化为1，dp[1]根据s[0..0]是不是合法的编码值而初始化为1或0 ，dp[i+1]的值由dp[i], dp[i-1], s[i-1..i-1], s[i-2..i-1]四个值决定，代码如下：

```java
	public int numDecodings(String s) {
		assert s != null && s.length() > 0;

		int len = s.length();

		int[] dp = new int[len + 1];
		dp[0] = 1;

		if (isValid(s.substring(0, 1))) {
			dp[1] = 1;
		} else {
			dp[1] = 0;
		}

		for (int i = 2; i <= len; i++) {
			String sub = s.substring(i-1, i);
			if (isValid(sub)) {
				dp[i] += dp[i-1];
			}

			sub = s.substring(i-2, i);
			if (isValid(sub)) {
				dp[i] += dp[i-2];
			}
		}

		return dp[len];
	}

	private boolean isValid(String s) {
		int len = s.length();

		if (len == 0 || len > 2) {
			return false;
		}

		int val = Integer.parseInt(s);
		if (len == 1 && val != 0) {
			return true;
		}
		if (len == 2 && val >= 10 && val <= 26) {
			return true;
		}

		return false;
	}
```

结果没有问题，但是时间效率较差，简单优化下：

```java
	public int numDecodings(String s) {
		assert s != null && s.length() > 0;

		int len = s.length();

		int[] dp = new int[len + 1];
		dp[0] = 1;

		if (isValid(s.substring(0, 1))) {
			dp[1] = 1;
		} else {
			return 0;
		}

		for (int i = 2; i <= len; i++) {
			String sub = s.substring(i-1, i);
			if (isValid(sub)) {
				dp[i] += dp[i-1];
			}

			sub = s.substring(i-2, i);
			if (isValid(sub)) {
				dp[i] += dp[i-2];
			}

			if (dp[i] == 0) {
				return 0;
			}
		}

		return dp[len];
	}
```

也就是说，一旦有某个子串的解码方式数量为0了，那么 整个串的解码方式数量也是0，直接返回就可以了。