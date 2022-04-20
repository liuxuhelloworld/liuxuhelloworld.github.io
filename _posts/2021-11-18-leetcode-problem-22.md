# 题目链接
https://leetcode-cn.com/problems/generate-parentheses/

# 解答过程
## 动态规划
这个题目从动态规划的角度来考虑，我觉得不容易。从代码来看，还是挺好理解的，但是我觉得，如果没有相关经验，并不容易想出状态转移方程。每一个合法的括号序列可以表示为**(a)b**，递归的a和b都是合法的括号序列（可以为空）。

```java
	public List<String> generateParenthesis(int n) {
		if (n <= 0) {
			return Collections.singletonList("");
		}

		if (n == 1) {
			return Collections.singletonList("()");
		}

		List<String>[] dp = new List[n+1];
		dp[0] = Collections.singletonList("");
		dp[1] = Collections.singletonList("()");
		for (int i = 2; i < dp.length; i++) {
			List<String> all = new ArrayList<>();
			for (int j = 0; j < i; j++) {
				List<String> parts1 = dp[j];
				List<String> parts2 = dp[i-1-j];

				for (String part1 : parts1) {
					for (String part2 : parts2) {
						all.add("(" + part1 + ")" + part2);
					}
				}
			}

			dp[i] = all;
		}

		return dp[n];
	}
```

## 回溯法
这个题目从回溯法的角度来考虑，从代码来看，倒是简洁很多。思路有一点讨巧，但是很好理解，很好的体现了回溯的思想。

```java
	public List<String> generateParenthesis(int n) {
		List<String> res = new ArrayList<>();

		backtrack(res, n, n, new String());

		return res;
	}

	private void backtrack(List<String> res, int openLeft, int closeLeft, String s) {
		if (openLeft == 0 && closeLeft == 0) {
			res.add(s);
			return;
		}

		if (openLeft > closeLeft) {
			return;
		}

		if (openLeft > 0) {
			backtrack(res, openLeft-1, closeLeft, s + "(");
		}

		if (closeLeft > 0) {
			backtrack(res, openLeft, closeLeft-1, s + ")");
		}
	}
```