# 题目链接
https://leetcode.cn/problems/smallest-integer-divisible-by-k/

# 解答过程
这个题目算是比较纯粹的数学题目了。题目本身不难理解，给定一个整数k，求完全由数字1组成的、能被k整除的、最小的那个整数包含的数字1的个数。

从简单直接的想法入手，我们可以由小到大挨个遍历1、11、111、...，直到遍历到能被k整数的那一个。但是这里面有两个问题：
- 题目明确说明不限于64位整数的范围，遍历一定次数后会overflow
- 如果对于给定的k，不存在满足题目要求的整数呢？也就是什么时候跳出遍历
  
对于第一个问题，某些语言可能能支持大数，比如Java中使用BigInteger，但这样未免过于取巧。那么应当如何避免溢出问题呢？对于题目而言，其实我们关心的是当前遍历到的完全由数字1组成的整数对k取余的结果，当取余为0时，说明我们找到了满足题目要求的整数。换句话说，我们不需要真的由小到大遍历每个完全由数字1组成的整数，而只需要维护这样的整数对k取余的结果即可。而根据数学运算，x1 % k == (x * 10 + 1) % k == (x % k) * 10 + 1

对于第二个问题，直接说答案吧，如果对于给定的k，不存在满足题目要求的整数，那么由小到大遍历完全由数字1组成的整数，并对k取余，这个余数一定是1到k-1中的一个，并且会按一定顺序挨个出现，直到重复。额，具体的数学推导我是不记得了，但这个结论还是有印象的，应该是和费马大定理有关。

另外，对于尾数是偶数或者是5的，可以直接返回，肯定不会存在满足题目要求的整数。

```java
	public int smallestRepunitDivByK(int k) {
		if (k == 1) {
			return 1;
		}

		if (k % 2 == 0) {
			return -1;
		}

		if (k % 5 == 0) {
			return -1;
		}

		int cnt = 1;
		int val = 1;
		while (true) {
			val = val*10 + 1;
			cnt++;

			int remainder = val % k;
			if (remainder % k == 0) {
				return cnt;
			}

			if (remainder == 1) {
				break;
			}

			val = remainder;
		}

		return -1;
	}
```