# 题目链接
https://leetcode.cn/problems/filling-bookcase-shelves/

# 解答过程
这个题目，坦白讲，光理解题目意图就花了一定的时间，但是总结起来也不复杂。给定一组书，每本书有宽度和高度两个属性，还有一个固定宽度的书架，现在要把这些书按给定的顺序摆放到书架上，摆不开了就换层，求能实现摆放完所有书的最小总层高是多少。这里面有一个关键点是要按给定的顺序摆放到书架上。从直觉上，要想让总层高最小，那么就应该尽量把高度大的书摆放到同一层，因为每一层的高度是由这一层中最高的书决定的，如果高度大的书摆放到了不同层，那么显然有悖于目标。但是由于限制要按给定的顺序摆放，不能调整书的顺序而把高度大的书尽量放到同一层，所以这就涉及到了在哪里去开始下一层的问题。

那么如何得到最小总层高呢？这种求最小值的问题，直觉上容易想到动态规划。动态规划的关键是由小到大的解决问题，入手点就是推理问题由小到大的过程中如何利用小规模问题的解得到大规模问题的解。

这个题目我们可以这样推理：处理第一本书，那么显然最小总层高就是这本书的高度，我们记为dp\[0\]。处理第二本书，如果不换层，那么总层高是第一本书和第二本书高度的较大值，如果换层，那么总层高是第一本书和第二本书高度之和，显然不换层是更好的选择，但是是不是可以不换层要看两本书的宽度是否满足书架宽度的限制，如果一层放不下，只能换层，如果一层能放下，显然就不换层，由此我们可以得到处理完第二本书后的最小总层高，我们记为dp\[1\]。处理第三本书，直接换层，加上dp\[1\]可得一层高；尝试将第二本书与第三本书摆放到同一层，加上dp\[0\]可得一层高；尝试将第二本书、第一本书与第三本书均摆放到同一层，可得一层高；这个过程均需验证是否满足书架宽度限制，由此我们可以得到处理完第三本书后的最小总层高，我们记为dp\[2\]。以此类推，处理到第i本书时，为了得到dp\[i-1\]，我们需要从第i本书倒序遍历，直到因书架宽度限制而不能与第i本书摆放到同一层的那一本，在这个倒序遍历过程中，我们需要计算每种可能下的层高，最终得到最小总层高dp\[i-1\]。

```java
	public int minHeightShelves(int[][] books, int shelfWidth) {
		int length = books.length;

		int[] thick = new int[length];
		int[] height = new int[length];

		for (int i = 0; i < length; i++) {
			thick[i] = books[i][0];
			height[i] = books[i][1];
		}

		// dp[i] represents min height to fill books[0..i]
		int[] dp = new int[length];
		dp[0] = height[0];

		for (int i = 1; i < length; i++) {
			int thickSum = 0;
			int maxBookHeight = height[i];
			int minShelfHeight = Integer.MAX_VALUE;
			for (int k = i; k >= 0; k--) {
				thickSum += thick[k];
				if (height[k] > maxBookHeight) {
					maxBookHeight = height[k];
				}

				if (thickSum <= shelfWidth) {
					if (k == 0) {
						if (maxBookHeight < minShelfHeight) {
							minShelfHeight = maxBookHeight;
						}
					} else {
						if (dp[k - 1] + maxBookHeight < minShelfHeight) {
							minShelfHeight = dp[k - 1] + maxBookHeight;
						}
					}
				}
			}

			dp[i] = minShelfHeight;
		}

		return dp[length-1];
	}
```

这不是我第一版的代码，写第一版代码时我使用了一个二维的dp数组，但是思路是一致的：

```java
	public int minHeightShelves(int[][] books, int shelfWidth) {
		int length = books.length;

		int[] thick = new int[length];
		int[] height = new int[length];

		for (int i = 0; i < length; i++) {
			thick[i] = books[i][0];
			height[i] = books[i][1];
		}

		// dp[i][j] represents min height to fill books[i..j]
		int[][] dp = new int[length][length];

		for (int i = 0; i < length; i++) {
			dp[i][i] = height[i];
		}

		for (int gap = 1; gap < length; gap++) {
			for (int i = 0; i < length - gap; i++) {
				int thickSum = 0;
				int maxBookHeight = height[i+gap];
				int minShelfHeight = Integer.MAX_VALUE;
				for (int k = i + gap; k >= i; k--) {
					thickSum += thick[k];
					if (height[k] > maxBookHeight) {
						maxBookHeight = height[k];
					}

					if (thickSum <= shelfWidth) {
						if (k == 0) {
							if (maxBookHeight < minShelfHeight) {
								minShelfHeight = maxBookHeight;
							}
						} else {
							if (dp[i][k-1] + maxBookHeight < minShelfHeight) {
								minShelfHeight = dp[i][k-1] + maxBookHeight;
							}
						}
					}
				}

				dp[i][i+gap] = minShelfHeight;
			}
		}

		return dp[0][length-1];
	}
```

第一版代码提交后，验证是通过的，但是效率较差，稍作分析就发现只需要一维数组即可，所以重写成第二版代码。

但是，第二版代码提交后，还是效率较差。。。

看了下官方题解，我觉得思路上有一定共通，但是具体计算过程没看懂，有时间再研究下。