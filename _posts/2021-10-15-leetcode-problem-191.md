# 题目链接
https://leetcode-cn.com/problems/number-of-1-bits/

# 解答过程
这个题目最笨的办法通过二进制移位循环处理都可以的，不过Java不支持无符号整数，所以对于负数应该得做左移，并且是和Integer.MIN_VALUE做二进制且操作来判断是否是二进制1。

对于整型编码具备一定认知的话，可以通过n & (n-1)来操作，每次n & (n-1)会抹掉低位的一个二进制1

```java
	public int hammingWeight(int n) {
		if (n == 0) {
			return 0;
		}

		int weight = 0;
		do {
			weight++;
		} while ((n = (n & n-1)) != 0);


		return weight;
	}
```