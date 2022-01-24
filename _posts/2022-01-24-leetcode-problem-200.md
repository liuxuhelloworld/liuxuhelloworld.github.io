# 题目链接
https://leetcode-cn.com/problems/number-of-islands/

# 解答过程
在不清楚题目标签的情况下，对于这个题目，我想到的是动态规划。。。，然后就写了一坨代码，对于题目示例的输入，还就是正确的。使用的状态转移方程是dp\[i\]\[\j]表示\[0\]\[0\]-\[i\]\[j\]这个子数组的岛屿数目，而dp\[i\]\[j\]的计算涉及dp\[i-1\]\[j\], dp\[i\]\[j-1\]和dp\[i-1\]\[j-1\]，其实从直觉上应该就能发现问题了，思路太复杂，很难从逻辑上验证它的正确性。后面看了官方题解才明白大的思路就错了。

从深度优先遍历的角度来考虑这个题目，思路就比较清晰了，以某一个节点为起始，按照一定顺序去深度优先遍历值为1的节点，并标注访问状态，直到没有能到达的值为1的节点结束，这样就等于把某一个连通的岛屿的所有节点全部访问到标注了访问状态。上层做二维数组遍历，碰到值为1的节点并且尚未标注访问状态的，说明碰到了新岛屿，那么从这个节点开始做深度优先遍历。

```java
	private int[] dx = {-1, 1, 0, 0};
	private int[] dy = {0, 0, -1, 1};

	public int numIslands(char[][] grid) {
		int m = grid.length;
		int n = grid[0].length;

		boolean[][] visited = new boolean[m][n];

		int num = 0;

		for (int i = 0; i < m; i++) {
			for (int j = 0; j < n; j++) {
				if (grid[i][j] == '1'
					&& !visited[i][j]) {
					num++;
					dfs(grid, i, j, visited);
				}
			}
		}

		return num;
	}

	private void dfs(char[][] grid, int i, int j, boolean[][] visited) {
		if (visited[i][j]
			|| grid[i][j] == '0') {
			return;
		}

		visited[i][j] = true;

		for (int k = 0; k < 4; k++) {
			int x = i + dx[k];
			int y = j + dy[k];

			if (isValidIndex(grid, x, y)
				&& grid[x][y] == '1'
				&& !visited[x][y] ) {
				dfs(grid, x, y, visited);
			}
		}
	}

	private boolean isValidIndex(char[][] grid, int i, int j) {
		int m = grid.length;
		int n = grid[0].length;

		if (i < 0 || i >= m) {
			return false;
		}

		if (j < 0 || j >= n) {
			return false;
		}

		return true;
	}
```