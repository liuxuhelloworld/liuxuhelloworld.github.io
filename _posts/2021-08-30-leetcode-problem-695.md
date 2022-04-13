# 题目链接
https://leetcode-cn.com/problems/max-area-of-island/

# 解答过程
## DFS
这个题目和Problem130、Problem200很类似，同样的DFS逻辑，只不过需要返回每次DFS遍历到了多少个值为1的元素。

```java
	int[] dx = new int[] {0, -1, 0, 1};
	int[] dy = new int[] {1, 0, -1, 0};

	public int maxAreaOfIsland(int[][] grid) {
		int m = grid.length;
		int n = grid[0].length;

		boolean[][] visited = new boolean[m][n];

		int maxArea = 0;
		for (int i = 0; i < m; i++) {
			for (int j = 0; j < n; j++) {
				if (grid[i][j] == 1
					&& visited[i][j] == false) {
					maxArea = Math.max(maxArea, dfs(grid, i, j, visited));
				}
			}
		}

		return maxArea;
	}

	private int dfs(int[][] grid, int curRow, int curCol, boolean[][] visited) {
		if (grid[curRow][curCol] == 0
			|| visited[curRow][curCol] == true) {
			return 0;
		}

		int area = 1;
		visited[curRow][curCol] = true;
		for (int i = 0; i < 4; i++) {
			int nextRow = curRow + dx[i];
			int nextCol = curCol + dy[i];
			if (nextRow >= 0 && nextRow < grid.length
				&& nextCol >= 0 && nextCol < grid[0].length) {
				area += dfs(grid, nextRow, nextCol, visited);
			}
		}

		return area;
	}
```

## BFS
BFS的解法也比较容易理解。从某个点开始，上下左右四个方向，联通的点入队列，同时，为了避免重复，借助一个visited的布尔数组。

```java
	int[] dx = new int[] {0, -1, 0, 1};
	int[] dy = new int[] {1, 0, -1, 0};

	public int maxAreaOfIsland(int[][] grid) {
		int maxArea = 0;

		int m = grid.length;
		int n = grid[0].length;

		Queue<int[]> queue = new LinkedList<>();

		boolean[][] visited = new boolean[m][n];

		for (int i = 0; i < m; i++) {
			for (int j = 0; j < n; j++) {
				if (grid[i][j] == 1 && visited[i][j] == false) {
					queue.clear();

					queue.add(new int[] {i, j});
					visited[i][j] = true;
					int area = 1;
					while (!queue.isEmpty()) {
						int[] cur = queue.poll();
						for (int k = 0; k < 4; k++) {
							int nextRow = cur[0] + dx[k];
							int nextCol = cur[1] + dy[k];
							if (nextRow >= 0 && nextRow < m
								&& nextCol >= 0 && nextCol < n
								&& grid[nextRow][nextCol] == 1
								&& visited[nextRow][nextCol] == false) {
								queue.add(new int[] {nextRow, nextCol});
								visited[nextRow][nextCol] = true;
								area++;
							}
						}
					}

					if (area > maxArea) {
						maxArea = area;
					}
				}
			}
		}

		return maxArea;
	}
```