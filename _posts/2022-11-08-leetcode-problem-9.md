# 题目链接
https://leetcode.cn/problems/palindrome-number/

# 解答过程
显然，如果入参是负数，那么就不是回文数。对于非负数，通过转换得到一个char数组的话，很容易判断是不是回文。但是题目里明确提示，有没有办法不做字符串转换？对于一个整数，从低位到高位遍历每一位数字是很容易的，那么如何在这样一个遍历过程中判断是否回文呢？我们在从低位到高位遍历的过程中，维护一个新的逆序的整数值，根据回文的特征，如果原整数是回文的，那么最终得到的新的逆序的整数和原整数应该相等。

```java
	public boolean isPalindrome(int x) {
		if (x < 0) {
			return false;
		}

		int y = x, reversed = 0;
		while (y != 0) {
			int remain = y % 10;
			reversed = reversed*10 + remain;
			y = y / 10;
		}

		return x == reversed;
	}
```

# 相似题目
- [Problem5-Longest Palindromic Substring](2021-10-25-leetcode-problem-5.md)
- [Problem7-Reverse Integer](2022-11-15-leetcode-problem-7.md)
- [Problem125-Valid Palindrome](2022-11-09-leetcode-problem-125.md)