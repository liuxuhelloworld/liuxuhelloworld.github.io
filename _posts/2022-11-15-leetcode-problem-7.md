# 题目链接
https://leetcode.cn/problems/reverse-integer/

# 解答过程
按十进制位反转一个整数的思路是比较直接的，由低位到高位，我们可以很方便地遍历一个整数的十进制位，在遍历过程中，也很容易维护一个反转后的整数值。唯一的难点是如何处理overflow的问题。如果可以使用长整型，那么overflow的问题也很容易解决。但是题目明确要求不要使用长整型。稍作思考，只能在可能overflow前去判断。

```java
	public int reverse(int x) {
		if (x == Integer.MIN_VALUE) {
			return 0;
		}

		int maxIntDivide10 = Integer.MAX_VALUE / 10;
		int maxIntDivide10Remain = Integer.MAX_VALUE % 10;

		boolean negative = x < 0;
		int absX = x >= 0 ? x : -x;

		int reverse = 0;
		while (absX > 0) {
			int remain = absX % 10;

			if (reverse > maxIntDivide10
				|| (reverse == maxIntDivide10 && remain > maxIntDivide10Remain)) {
				return 0;
			}

			reverse = reverse * 10 + remain;

			absX = absX / 10;
		}

		return negative ? -reverse : reverse;
	}
```

# 相似题目
- [Problem9-Palindrome Number](2022-11-08-leetcode-problem-9.md)