# 题目链接
https://leetcode-cn.com/problems/reverse-string/

# 解答过程
额， 这个题目打双指针的标签是不是有点勉强......
```java
	public void reverseString(char[] s) {
		for (int i = 0, j = s.length - 1; i < j; i++, j--) {
			char tmp = s[i];
			s[i] = s[j];
			s[j] = tmp;
		}
	}
```