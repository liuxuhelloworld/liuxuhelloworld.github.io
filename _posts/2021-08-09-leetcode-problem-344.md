# 题目链接
https://leetcode-cn.com/problems/reverse-string/

# 解答过程
额， 这个题有点简单的不像话了。
```java
	public void reverseString(char[] s) {
		for (int i = 0, j = s.length - 1; i < j; i++, j--) {
			char tmp = s[i];
			s[i] = s[j];
			s[j] = tmp;
		}
	}
```