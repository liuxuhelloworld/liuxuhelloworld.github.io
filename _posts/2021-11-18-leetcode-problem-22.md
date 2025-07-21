# 题目链接
https://leetcode-cn.com/problems/generate-parentheses/

# 解答过程
这个题目需要计算n组括号组成的字符串中，满足括号开闭语法规则的字符串。最简单直接的做法就是遍历所有可能组成的字符串，并判断每个字符串是否满足开闭语法规则。这种遍历所有可能，并过滤合法解，已经是使用回溯法的强暗示。从代码来看，非常简洁，通过判断开闭括号剩余数的关系完成剪枝，通过控制开闭括号的添加顺序保证满足语法规则。思路有一点讨巧，但是很好理解，同时很好地体现了回溯的思想。

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