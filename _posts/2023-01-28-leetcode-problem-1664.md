# 题目链接
https://leetcode.cn/problems/ways-to-make-a-fair-array/

# 解答过程
题目本身很容易理解，去掉数组中的一个元素，剩余元素按奇偶位置取和，取和相等的称为pair. 给定一个数组，计算有多少个位置满足pair。我往往是先从暴力解法入手，对于这个题目，暴力解法很简单，挨个位置遍历嘛，遍历每个位置时计算剩余元素的奇偶位置和。当然，不需要真的移动被去掉元素后面的元素的位置，因为很容易想明白，假设去掉某个位置的元素，后面元素向前补一位，那么这些补位的元素的位置的奇偶性会反转，即原来是奇数位置的会变成偶数位置，原来是偶数位置的会变成奇数位置。暴力解法的思路捋清楚以后，就要想如何优化，着手点就是看看哪里有重复计算。这个题目还是比较容易看出暴力解法的重复计算在哪的，暴力解法在遍历每个位置时都是独立的，没有利用上前序位置已经计算出的结果，即处理第i+1个位置时没有利用上第i个位置的计算结果。那么能不能利用上呢？我们定义四个变量，leftOddSum, leftEvenSum, rightOddSum, rightEvenSum，循环不变量是遍历到第i个位置时，leftOddSum和leftEvenSum表示第i个位置前（不包含第i个）的子序列分别的奇偶和，rightOddSum和rightEvenSum表示从第i个位置开始（包含第i个）的子序列分别的奇偶和。那么遍历到第i个位置表示要去除第i个元素，所以先根据第i个位置的奇偶性更新rightOddSum或rightEvenSum，然后比较leftOddSum和rightEvenSum的和是否与leftEvenSum和rightOddSum的和相等，如果相等，说明找到了一个pair位置。然后，在进入下一次循环前，将当前第i个位置的元素更新到leftOddSum或leftEvenSum.

```java
	public int waysToMakeFair(int[] nums) {
		int cnt = 0;

		int leftOddSum = 0, leftEvenSum = 0;
		int rightOddSum = 0, rightEvenSum = 0;
		for (int i = 0; i < nums.length; i++) {
			if (i % 2 == 0) {
				rightEvenSum += nums[i];
			} else {
				rightOddSum += nums[i];
			}
		}

		for (int i = 0; i < nums.length; i++) {
			boolean curIdxEven = i % 2 == 0;

			if (curIdxEven) {
				rightEvenSum -= nums[i];
			} else {
				rightOddSum -= nums[i];
			}

			if (leftEvenSum + rightOddSum == leftOddSum + rightEvenSum) {
				cnt++;
			}

			if (curIdxEven) {
				leftEvenSum += nums[i];
			} else {
				leftOddSum += nums[i];
			}
		}

		return cnt;
	}
```

看了下官方题解，解法是差不多的，不过官方题解是从动态规划方程入手，更数学一些，值得学习。