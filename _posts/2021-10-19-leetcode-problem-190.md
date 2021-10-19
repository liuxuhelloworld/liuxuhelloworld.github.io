# 题目链接
https://leetcode-cn.com/problems/reverse-bits/

# 解答过程
通过右移循环取出最低位的二进制位，然后通过左移累积返回值即可。

```java
	public int reverseBits(int n) {
		int ret = 0;

		for (int i = 0; i < 32; i++) {
			int rightest = n & 1;
			n = n >> 1;
			ret = ret << 1;
			ret = ret | rightest;
		}

		return ret;
	}
```