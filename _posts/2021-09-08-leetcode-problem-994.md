# 题目链接
https://leetcode-cn.com/problems/rotting-oranges/

# 解答过程
这个题目其实和Problem542就很像了，按分钟rotten就可以看作计算从rottened点到fresh点的距离，先把所有rottened的点看作一个整体入队列，然后每次循环处理时，把作为整体入队列的点当作一批来处理，直到队列为空。

```java
	private int[] dx = new int[] {0, -1, 0, 1};
	private int[] dy = new int[] {1, 0, -1, 0};

	public int orangesRotting(int[][] grid) {
		int m = grid.length;
		int n = grid[0].length;

		Queue<int[]> queue = new LinkedList<>();

		for (int i = 0; i < m; i++) {
			for (int j = 0; j < n; j++) {
				if (grid[i][j] == 2) {
					queue.add(new int[] {i, j});
				}
			}
		}

		int minutes = 0;
		while (!queue.isEmpty()) {
			int size = queue.size();
			boolean flag = false;
			while (size-- > 0) {
				int[] cur = queue.poll();
				int curRow = cur[0], curCol = cur[1];
				for (int i = 0; i < 4; i++) {
					int nextRow = curRow + dx[i];
					int nextCol = curCol + dy[i];
					if (nextRow >= 0 && nextRow < m
						&& nextCol >= 0 && nextCol < n
						&& grid[nextRow][nextCol] == 1) {
						grid[nextRow][nextCol] = 2;
						queue.add(new int[] {nextRow, nextCol});
						flag = true;
					}
				}
			}

			if (flag) {
				minutes++;
			}
		}

		for (int i = 0; i < m; i++) {
			for (int j = 0; j < n; j++) {
				if (grid[i][j] == 1) {
					return -1;
				}
			}
		}

		return minutes;
	}
```