# 题目链接
https://leetcode-cn.com/problems/single-number/

# 解答过程
利用v ^ v = 0的特性，很容易想到：

```java
	public int singleNumber(int[] nums) {
		int ret = 0;

		for (int num : nums) {
			ret ^= num;
		}

		return ret;
	}
```	