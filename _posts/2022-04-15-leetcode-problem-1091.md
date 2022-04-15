# 题目链接
https://leetcode-cn.com/problems/shortest-path-in-binary-matrix/

# 解答过程
这个题目和Probem542、Problem994很类似，稍有区别就是这个题目是可以斜着遍历，即可以以相邻点做遍历，而不限于相邻边。简单来说就是从源点开始按宽度做扩散，每次处理一批距离源点相同宽度的0点，判断这些0点中是否已经扩散到了终点，如果是，返回path长度。如果不是，则把下一宽度能扩散到的0点入队列，并更新path。如果遍历结束都没有扩散到终点，则没有clear path，返回-1。提交以后是通过的，不过效率较差，有时间再研究下效率问题。

```java
	private int[] dx = new int[] {-1, -1, -1, 0, 1, 1, 1, 0};
	private int[] dy = new int[] {-1, 0, 1, 1, 1, 0, -1, -1};

	public int shortestPathBinaryMatrix(int[][] grid) {
		int n = grid.length;

		if (grid[0][0] != 0 || grid[n-1][n-1] != 0) {
			return -1;
		}

		boolean[][] visited = new boolean[n][n];

		Queue<int[]> queue = new LinkedList<>();
		queue.add(new int[] {0, 0});
		visited[0][0] = true;

		int path = 1;
		while (!queue.isEmpty()) {
			int size = queue.size();
			boolean flag = false;
			for (int i = 0; i < size; i++) {
				int[] cur = queue.poll();
				int row = cur[0], col = cur[1];
				if (row == n-1 && col == n-1) {
					return path;
				}

				for (int k = 0; k < dx.length; k++) {
					int newRow = row + dx[k];
					int newCol = col + dy[k];

					if (isValid(newRow, newCol, n)
						&& grid[newRow][newCol] == 0
						&& !visited[newRow][newCol]) {
						flag = true;
						queue.add(new int[] {newRow, newCol});
						visited[newRow][newCol] = true;
					}
				}
			}

			if (flag) {
				path++;
			}
		}

		return -1;
	}

	private boolean isValid(int row, int col, int n) {
		if (row < 0 || row >= n) {
			return false;
		}

		if (col < 0 || col >= n) {
			return false;
		}

		return true;
	}
```