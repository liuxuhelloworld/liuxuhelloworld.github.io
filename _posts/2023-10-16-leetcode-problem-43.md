# 题目链接
https://leetcode.cn/problems/multiply-strings/

# 解答过程
这个题目乍一看有点难度，我能想到的就是模拟竖式乘法，把字符串整数相乘拆解为字符串整数和数字相乘，然后进行字符串相加。在这个过程当中，要注意右侧补0等问题。

这个题目可以看作Problem415的升级版，Problem415是字符串整数相加，这里改为相乘。

简单看了下官方题解，官方题解给出的另外两种解法当然都更高级一些，尤其是卷积法。这些解法，如果没有相关经验或相关提示，并且用纸笔写写画画做推导，我觉得是不容易写出来的，有时间再研究下。

```java
	public String multiply(String num1, String num2) {
		int len1 = num1.length();
		int len2 = num2.length();

		if (len1 == 0 || len2 == 0) {
			throw new IllegalArgumentException();
		}

		String multiplicand, multiplier;
		if (len1 < len2) {
			multiplicand = num2;
			multiplier = num1;
		} else {
			multiplicand = num1;
			multiplier = num2;
		}

		if (multiplier.equals("0") || multiplicand.equals("0")) {
			return "0";
		}

		String leftShift = "";
		String result = "";
		for (int i = multiplier.length()-1; i >= 0; i--) {
			int ch = multiplier.charAt(i) - '0';

			result = addTwoStrings(result, multiplyByDigit(multiplicand, ch) + leftShift);
			leftShift += "0";
		}

		return result;
	}

	private String multiplyByDigit(String num, int digit) {
		if (!(digit >= 1 && digit <= 9)) {
			return "";
		}

		StringBuilder builder = new StringBuilder();
		int carry = 0;
		for (int i = num.length() - 1; i >= 0; i--) {
			int ch = num.charAt(i) - '0';

			int product = ch * digit + carry;
			int remainder = product % 10;
			carry = product / 10;

			builder.append((char)(remainder + '0'));
		}
		if (carry != 0) {
			builder.append((char)(carry + '0'));
		}

		return builder.reverse().toString();
	}

	private String addTwoStrings(String num1, String num2) {
		StringBuilder builder = new StringBuilder();

		String reversedNum1 = new StringBuilder(num1).reverse().toString();
		String reversedNum2 = new StringBuilder(num2).reverse().toString();

		int len1 = reversedNum1.length();
		int len2 = reversedNum2.length();

		int i, j;
		int carry = 0;
		for (i = 0, j = 0; i < len1 || j < len2; i++, j++) {
			int ch1, ch2;

			if (i < len1) {
				ch1 = reversedNum1.charAt(i) - '0';
			} else {
				ch1 = 0;
			}
			if (j < len2) {
				ch2 = reversedNum2.charAt(j) - '0';
			} else {
				ch2 = 0;
			}

			int sum = ch1 + ch2 + carry;

			int remainder = sum % 10;
			carry = sum / 10;
			builder.append((char)(remainder + '0'));
		}

		if (carry != 0) {
			builder.append((char)(carry + '0'));
		}

		return builder.reverse().toString();
	}
```

# 相似题目
- [Problem2-Add Two Numbers](2022-10-21-leetcode-problem-2.md)
- [Problem445-Two Sum](2022-10-27-leetcode-problem-445.md)
- [Problem415-Add Strings](2023-07-17-leetcode-problem-415.md)