# 题目链接
https://leetcode.cn/problems/equal-row-and-column-pairs/

# 解答过程
题目本身并不难理解，给定一个n*n的二维数组，按行列去找相等的pair，换句话说，用每一行和每一列来组合并判断是否相等。按照这样的思路，直接双重循环就可以了。判断某一行和某一列是否相等时，注意early return，碰到第一个不相等的元素即可返回，想来效率不会太差。但是，考虑到题目难度设置是中等，是不是有更高效的解法呢？思考了下，实在没有思路，先写个暴力解法的再说。

```java
	public int equalPairs(int[][] grid) {
		int length = grid.length;

		int cnt = 0;
		for (int i = 0; i < length; i++) {
			for (int j = 0; j < length; j++) {
				if (isEqual(grid, i, j)) {
					cnt++;
				}
			}
		}

		return cnt;
	}

	private boolean isEqual(int[][] grid, int rowIdx, int columnIdx) {
		int[] row = grid[rowIdx];
		for (int i = 0; i < row.length; i++) {
			if (row[i] != grid[i][columnIdx]) {
				return false;
			}
		}

		return true;
	}
```

代码提交以后没有问题，而且效率也还可以。看了下官方题解，其中对行和列做某种形式转换再基于哈希表做查询计数的思路有一定启发性，有时间再研究下。