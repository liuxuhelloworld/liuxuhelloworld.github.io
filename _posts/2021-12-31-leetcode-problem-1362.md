# 题目链接
https://leetcode-cn.com/problems/closest-divisors/

# 解答过程
额，面对这个题目我想到的就是暴力解法，抽象出一个计算某个数绝对值差最小的乘数对的方法，然后分别计算num+1和num+2的绝对值差最小的乘数对，然后比较它们哪个绝对值差更小。

```java
	public int[] closestDivisors(int num) {
		int target1 = num + 1;
		int target2 = num + 2;

		int[] divisors1 = minDiffDivisors(target1);
		int[] divisors2 = minDiffDivisors(target2);

		if (Math.abs(divisors1[0] - divisors1[1]) < Math.abs(divisors2[0] - divisors2[1])) {
			return divisors1;
		} else {
			return divisors2;
		}
	}

	private int[] minDiffDivisors(int num) {
		int max = (int) Math.floor(Math.sqrt(num));

		int minDiff = Integer.MAX_VALUE;
		int[] ret = new int[2];
		for (int i = 1; i <= max; i++) {
			if (num % i == 0) {
				int j = num / i;
				int curDiff = Math.abs(i - j);
				if (curDiff < minDiff) {
					minDiff = curDiff;
					ret[0] = i;
					ret[1] = j;
				}
			}
		}

		return ret;
	}
```

提交以后是没有问题的，不过效率是差了点。额，看了下官方题解，基本是一样的思路。