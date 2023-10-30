# 题目链接
https://leetcode.cn/problems/valid-palindrome/

# 解答过程
这个题目确实是比较简单了，基本的字符串处理操作，或者先做过滤和转换，或者直接由两端向中间遍历，逻辑都差不多，不赘述。

```java
	public boolean isPalindrome(String s) {
		StringBuilder builder = new StringBuilder();

		for (int i = 0; i < s.length(); i++) {
			char ch = s.charAt(i);
			if ((ch >= 'a' && ch <= 'z')
				|| (ch >= '0' && ch <= '9')) {
				builder.append(ch);
			} else if (ch >= 'A' && ch <= 'Z') {
				builder.append((char)(ch - 'A' + 'a'));
			}
		}

		char[] chars = builder.toString().toCharArray();
		for (int i = 0, j = chars.length - 1; i < j; i++, j--) {
			if (chars[i] != chars[j]) {
				return false;
			}
		}

		return true;
	}
```

# 相似题目
- [Problem5-Longest Palindromic Substring](2021-10-25-leetcode-problem-5.md)
- [Problem9-Palindrome Number](2022-11-08-leetcode-problem-9.md)
- [Problem409-Longest Palindrome](2021-12-24-leetcode-problem-409.md)