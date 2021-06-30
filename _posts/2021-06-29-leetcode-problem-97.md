# 题目链接
https://leetcode-cn.com/problems/interleaving-string/

# 解答过程
一开始面对这个题，就感觉好难的样子，稍微想了一下，有了一个递归的思路，交替使用s1和s2去和s3匹配最左子串，如果匹配，截掉最左子串，并且交换s1和s2，则得到了一个更小规模的子问题，代码如下：

```java
	public boolean isInterleave(String s1, String s2, String s3) {
		assert s1 != null && s2 != null && s3 != null;

		if (s1.equals("") && s2.equals("") && s3.equals("")) {
			return true;
		}

		if (matches(s1, s3) > 0) {
			boolean isInterleave = recur(s1, s2, s3);
			if (isInterleave) {
				return true;
			}
		}

		if (matches(s2, s3) > 0) {
			return recur(s2, s1, s3);
		}

		return false;
	}

	private boolean recur(String start, String other, String target) {
		if (start.equals("")
			&& other.equals("")
			&& target.equals("")) {
			return true;
		}

		int matches = matches(start, target);
		if (matches == 0) {
			return false;
		}

		return recur(other, start.substring(matches), target.substring(matches));
	}

	private int matches(String s, String t) {
		int cnt = 0;
		for (int i = 0, j = 0; i < s.length() && j < t.length(); i++, j++) {
			if (s.charAt(i) == t.charAt(j)) {
				cnt++;
			} else {
				break;
			}
		}

		return cnt;
	}
```
写完这个自己还挺得意的，看起来是比较简洁地解决了问题，用几个简单的case测了一下也没有问题。但是提交以后就报解答错误了，错误的case是：s1 = "aabcc", s2 = "dbbca", s3 = "aadbcbbcac"，为什么答案错了呢？研究一下这个case就能发现，上面截掉最左子串的处理使得某些interleave没有覆盖到，由此，优化如下：

```java
	public boolean isInterleave(String s1, String s2, String s3) {
		assert s1 != null && s2 != null && s3 != null;

		if (s1.equals("") && s2.equals("") && s3.equals("")) {
			return true;
		}

		if (matches(s1, s3) > 0) {
			boolean isInterleave = recurV2(s1, s2, s3);
			if (isInterleave) {
				return true;
			}
		}

		if (matches(s2, s3) > 0) {
			return recurV2(s2, s1, s3);
		}

		return false;
	}

	private boolean recurV2(String start, String other, String target) {
		if (start.equals("")
			&& other.equals("")
			&& target.equals("")) {
			return true;
		}

		int matches = matches(start, target);
		if (matches == 0) {
			return false;
		}

		for (int i = 1; i < matches; i++) {
			if (!other.isEmpty()
				&& start.charAt(i) == other.charAt(0)) {
				boolean ret = recurV2(other, start.substring(i), target.substring(i));
				if (ret) {
					return true;
				}
			}
		}

		return recurV2(other, start.substring(matches), target.substring(matches));
	}

	private int matches(String s, String t) {
		int cnt = 0;
		for (int i = 0, j = 0; i < s.length() && j < t.length(); i++, j++) {
			if (s.charAt(i) == t.charAt(j)) {
				cnt++;
			} else {
				break;
			}
		}

		return cnt;
	}
```

这种处理结果应该是没问题的，但是提交以后超时。。。

其实到这已经比较明显了，递归超时，应该考虑用动态规划了。

```java
	public boolean isInterleaveV2(String s1, String s2, String s3) {
		assert s1 != null && s2 != null && s3 != null;

		int len1 = s1.length();
		int len2 = s2.length();
		int len3 = s3.length();

		if (len1 + len2 != len3) {
			return false;
		}

		if (len1 == 0 && len2 == 0 && len3 == 0) {
			return true;
		}

		boolean[][] dp = new boolean[len1 + 1][len2 + 1];

		int matches1 = matches(s1, s3);
		for (int len = 1; len <= matches1; len++) {
			dp[len][0] = true;
		}

		int matches2 = matches(s2, s3);
		for (int len = 1; len <= matches2; len++) {
			dp[0][len] = true;
		}

		for (int sum = 2; sum <= len1 + len2; sum++) {
			for (int i = 1; i <= sum; i++) {
				int j = sum - i;

				if (i <= 0 || i > len1 || j <= 0 || j > len2) {
					continue;
				}

				if (s1.charAt(i - 1) == s3.charAt(i + j - 1)) {
					dp[i][j] = dp[i][j] || dp[i - 1][j];
				}
				if (s2.charAt(j - 1) == s3.charAt(i + j - 1)) {
					dp[i][j] = dp[i][j] || dp[i][j - 1];
				}
			}
		}

		return dp[len1][len2];
	}
	
		private int matches(String s, String t) {
		int cnt = 0;
		for (int i = 0, j = 0; i < s.length() && j < t.length(); i++, j++) {
			if (s.charAt(i) == t.charAt(j)) {
				cnt++;
			} else {
				break;
			}
		}

		return cnt;
	}
```

这次没有问题了，虽然效率还是比较差的只击败了17%的提交。

这个题前后花了不少时间，在这种题的直觉上总是有点问题。