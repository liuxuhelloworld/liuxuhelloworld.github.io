# 题目链接
https://leetcode.cn/problems/alternating-digit-sum/

# 解答过程
这个题目比较简单了，稍微麻烦一点的地方是通过取余计算得到每一位数字的顺序和正负符号的赋值顺序相反，通过一个栈暂存下即可。

```java
	public int alternateDigitSum(int n) {
		Stack<Integer> stack = new Stack<>();
		while (n != 0) {
			int remian = n % 10;
			stack.push(remian);
			n = n / 10;
		}

		int sign = 1;
		int sum = 0;
		while (!stack.isEmpty()) {
			int val = stack.pop();
			sum += sign * val;
			sign *= -1;
		}

		return sum;
	}
```