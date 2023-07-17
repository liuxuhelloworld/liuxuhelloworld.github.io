# 题目链接
https://leetcode.cn/problems/add-strings/

# 解答过程
这个题目主要考察基本的字符串操作能力，比较简单了，从题目要求以及参数限制来看，显然转换为整数再相加是行不通的，况且，不使用库函数的话，字符串转整数比这个题目本身还要难一些。这个题目就是逆序遍历字符串，挨个位置相加，注意处理进位即可。

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