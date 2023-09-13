# 题目链接
https://leetcode.cn/problems/check-knight-tour-configuration/

# 解答过程
这个题目乍一看描述有点不好理解，但结合示例就比较清楚了，简单说就是给定一个二维数组，每个元素的值表示第X步，注意走一步的规则是行列坐标加减2和加减1的组合，即下一步最多可能有8个备选位置，为什么是最多呢？因为可能有一些已经超出二维数组的合法下标范围。问题来了，对于给定的走法，是不是一个有效的能遍历所有位置一遍的走法呢？

我的解题思路比较直接，既然是判断给定的走法是不是有效的走法，那就模拟着走一遍嘛。起始位置是\[0,0\]，那么grid\[0\]\[0\]应当是0，然后我们计算以\[0,0\]为起点的下一个能走到的位置集合，即\{\[2,1\],\[1,2\]\}，遍历这个集合，看看有没有哪一个位置的值是1，如果有，那么说明这一步是可以有效的走下去。现在起始位置变成了新位置，再继续上面这个处理。如此往复，直到遍历完所有位置。

思路和代码都不复杂，效率上嘛，也就是O(n^2)，考虑到问题难度是中等，我还隐约觉得会不会超出时间限制。代码提交以后，证明我想多了......

```java
	public boolean checkValidGrid(int[][] grid) {
		int n = grid.length;

		int i = 0, j = 0;
		if (grid[i][j] != 0) {
			return false;
		}

		for (int target = 1; target < n*n; target++) {
			List<int[]> nextPosList = nextPosList(i, j, n);
			if (nextPosList.isEmpty()) {
				return false;
			}

			int[] validNexPos = validNextPos(nextPosList, grid, target);
			if (validNexPos == null) {
				return false;
			}

			i = validNexPos[0];
			j = validNexPos[1];
		}

		return true;
	}

	private List<int[]> nextPosList(int i, int j, int n) {
		List<int[]> list = new ArrayList<>();

		int nextI = i + 2;
		int nextJ = j + 1;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}
		nextJ = j - 1;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}

		nextI = i + 1;
		nextJ = j + 2;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}
		nextJ = j - 2;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}

		nextI = i - 1;
		nextJ = j + 2;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}
		nextJ = j - 2;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}

		nextI = i - 2;
		nextJ = j + 1;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}
		nextJ = j - 1;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}

		return list;
	}

	private boolean validPos(int i, int j, int n) {
		if (i < 0 || i >= n) {
			return false;
		}
		if (j < 0 || j >= n) {
			return false;
		}
		return true;
	}

	private int[] validNextPos(List<int[]> nextPosList, int[][] grid, int target) {
		for (int[] nextPos : nextPosList) {
			int i = nextPos[0];
			int j = nextPos[1];

			if (grid[i][j] == target) {
				return nextPos;
			}
		}

		return null;
	}
```