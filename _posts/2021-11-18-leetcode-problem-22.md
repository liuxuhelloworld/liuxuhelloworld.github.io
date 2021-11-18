# 题目链接
https://leetcode-cn.com/problems/generate-parentheses/

# 解答过程
这道题目也是以前写的了，但是很惭愧，乍一看还是没思路。。。印象中当时应该也是参考官方题解写的。

## 解法1
动态规划的写法，从代码来看，还是挺好理解的，但是我觉得，如果没有相关经验，想出状态转移方程并不容易，子问题的解双循环，把其中一个套起来，再和另一个拼接，额，我觉得是有点费劲。关键这种解法，我觉得背下来都不容易。。。

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

## 解法2
相反的，我觉得深度优先遍历的写法反而容易背下来。思路虽然有一点讨巧，但是不难理解，很容易给人一种，“对哦，就是这样”的感觉。

```java
	public List<String> generateParenthesis(int n) {
		List<String> res = new ArrayList<>();

		dfs(res, n, n, new String());

		return res;
	}

	private void dfs(List<String> res, int openLeft, int closeLeft, String s) {
		if (openLeft == 0 && closeLeft == 0) {
			res.add(s);
			return;
		}

		if (openLeft > closeLeft) {
			return;
		}

		if (openLeft > 0) {
			dfs(res, openLeft-1, closeLeft, s + "(");
		}

		if (closeLeft > 0) {
			dfs(res, openLeft, closeLeft-1, s + ")");
		}
	}
```