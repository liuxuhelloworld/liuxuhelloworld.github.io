# 题目链接

https://leetcode-cn.com/problems/integer-to-roman/

# 解法1
题目中限制入参在1到3999之间，romanSymbol是个十百千各位上0-9对应的罗马符号，然后从高位到低位循环处理入参即可。

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

# 解法2

解法1的可扩展性很差，把目标数值和对应的罗马字符串做成两个一一对应的数组，从左到右顺序处理即可。

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

