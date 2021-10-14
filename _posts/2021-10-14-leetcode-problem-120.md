# 题目链接
https://leetcode-cn.com/problems/triangle/

# 解答过程
从动态规划的角度去考虑，并不难想到如何解决这个问题。维护一个二维数组，dp\[i\]\[j]表示从top节点到第i行第j列的节点的最小路径sum，而dp\[i\]\[j\] = min(dp\[i-1\]\[j-1\], dp\[i-1\]\[j\]) + triangle\[i\]\[j\]，这样从二维数组的最后一行中找到最小值就是从top节点到最后一行的最小路径sum. 题目中进一步提到是不是可以只使用O(n)的额外空间，其实想一下，dp\[i\]这一行的计算只依赖dp\[i-1\]这一行，所以是可以只维护一个一维数组，这个数组存储的是从top节点到当前遍历到的那一行对应的各个元素的最小路径sum. 进一步的，遍历下一行时如何更新这个一维数组呢？dp\[i\]\[j\] = min(dp\[i-1\]\[j-1\], dp\[i-1\]\[j\]) + triangle\[i\]\[j\]，所以只能从右到左进行更新，从左到右更新会覆盖掉还需要进一步使用的原数据。

```java
	public int minimumTotal(List<List<Integer>> triangle) {
		int rows = triangle.size();

		int[] dp = new int[rows + 1];
		for (int i = 0; i <= rows; i++) {
			dp[i] = Integer.MAX_VALUE;
		}

		List<Integer> first = triangle.get(0);
		dp[1] = first.get(0);

		for (int i = 1; i < rows; i++) {
			List<Integer> cur = triangle.get(i);

			for (int j = i + 1; j >= 1; j--) {
				dp[j] = Math.min(dp[j], dp[j-1]) + cur.get(j-1);
			}
		}

		int min = dp[1];
		for (int i = 1; i <= rows; i++) {
			if (dp[i] < min) {
				min = dp[i];
			}
		}

		return min;
	}
```

注意dp数组的初始化逻辑和triangle第一行的处理。