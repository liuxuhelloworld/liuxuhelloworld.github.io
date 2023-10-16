# 题目链接
https://leetcode.cn/problems/add-strings/

# 解答过程
这个题目主要考察基本的字符串操作能力，比较简单了，从题目要求以及参数限制来看，显然转换为整数再相加是行不通的，况且，不使用库函数的话，字符串转整数比这个题目本身还要难一些。这个题目就是逆序遍历字符串，挨个位置相加，注意处理进位即可。

这个题目和Problem2、Problem445相比，最大的不同之处就在于字符串是可以随机存取的，无论顺序遍历还是逆序遍历，都好处理。而链表就只方便正序遍历，不方便逆序遍历，这也是Problem2和Problem445的最大区别。

```java
	public String addStrings(String num1, String num2) {
		StringBuilder builder = new StringBuilder();

		int len1 = num1.length();
		int len2 = num2.length();

		int carry = 0;
		for (int i = len1 - 1, j = len2 - 1; i >= 0 || j >= 0; i--, j--) {
			int val1 = 0;
			if (i >= 0) {
				val1 = num1.charAt(i) - '0';
			}
			int val2 = 0;
			if (j >= 0) {
				val2 = num2.charAt(j) - '0';
			}

			int sum = val1 + val2 + carry;
			int remain = sum % 10;
			carry = sum / 10;

			builder.append((char) (remain + '0'));
		}

		if (carry == 1) {
			builder.append("1");
		}

		return builder.reverse().toString();
	}
```

# 相似题目
- [Problem2-Add Two Numbers](2022-10-21-leetcode-problem-2.md)
- [Problem445-Two Sum](2022-10-27-leetcode-problem-445.md)
- [Problem43-Multiply Strings](2023-10-16-leetcode-problem-43.md)