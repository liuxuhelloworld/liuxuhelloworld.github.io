# 题目链接
https://leetcode-cn.com/problems/power-of-two/

# 解答过程
这个题目和Prolem191就类似了，利用n & (n-1)很容易判断是否是2的幂次。

```java
	public boolean isPowerOfTwo(int n) {
		return n > 0 && (n & (n-1)) == 0;
	}
```