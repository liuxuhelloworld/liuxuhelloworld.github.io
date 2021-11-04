# 题目链接
https://leetcode-cn.com/problems/integer-to-roman/

# 解答过程
## 解法1
这个题目最简单的思路就是从千位到个位挨个处理，拼接每一位对应的罗马字符就可以了：

```java
	private static String[][] romanSymbol = {
		{"", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"},
		{"", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC"},
		{"", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"},
		{"", "M", "MM", "MMM"}
	};

	public String intToRoman(int num) {
		StringBuilder builder = new StringBuilder();

		int op = 1000;
		while (num != 0) {
			int quotient = num / op;
			int remainder = num % op;

			if (quotient != 0) {
				builder.append(romanSymbol[(int)Math.log10(op)][quotient]);
			}

			op = op / 10;
			num = remainder;
		}

		return builder.toString();
	}
```

## 解法2
官方题解的思路更高级一些，其实就是把所有具备独特罗马字符表达的整数值罗列出来，然后把它们一般性地当作基数，进行一个常规的求商取余操作就可以了：

```java
	private static int[] val = {1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1};
	private static String[] symbol = {"M", "CM", "D", "CD", "C", "XC", "L", "XL", "X", "IX", "V", "IV", "I"};

	public String intToRomanV2(int num) {
		StringBuilder builder = new StringBuilder();

		int i = 0;
		while (num != 0) {
			int quotient = num / val[i];
			if (quotient != 0) {
				for (int j = 0; j < quotient; j++) {
					builder.append(symbol[i]);
				}
			}
			num = num % val[i];
			i++;
		}

		return builder.toString();
	}
```