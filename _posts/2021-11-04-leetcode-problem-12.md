# 题目链接
https://leetcode-cn.com/problems/integer-to-roman/

# 解答过程
题目乍一看有点唬人，简单来说，就是将阿拉伯数字转换为罗马数字。那么很直接的，按照罗马数字的进制规则，由大到小的处理基数就可以了，即按照1000、900、500、400、100、90、50、40、10、9、5、4、1的顺序，分别做如下计算：用当前值和基数做整数除法，计算得出包含的基数个数，添加到罗马数字表达法中，然后用余数和下一个基数重复此过程。

```java
	private static int[] val = {1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1};
	private static String[] symbol = {"M", "CM", "D", "CD", "C", "XC", "L", "XL", "X", "IX", "V", "IV", "I"};

	public String intToRoman(int num) {
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