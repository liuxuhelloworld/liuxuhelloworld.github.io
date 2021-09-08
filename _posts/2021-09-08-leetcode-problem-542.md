# 题目链接
https://leetcode-cn.com/problems/01-matrix/

# 解答过程
这个题目是看了官方题解以后写的，看了官方题解后才有感觉，把所有为0的点看作一个整体，进行BFS，想一想是很合理，也符合直觉的想法嘛。自己一开始还是想用DFS，很快就陷入了双向依赖的死循环；也有考虑动态规划，但想的是怎么设计一个统一的动态规划方程，而实际只能是四个方向分别做动态规划。

```java
	int[] dx = new int[] {0, -1, 0, 1};
	int[] dy = new int[] {1, 0, -1, 0};

	public int[][] updateMatrix(int[][] mat) {
		int m = mat.length;
		int n = mat[0].length;

		Queue<int[]> queue = new LinkedList<>();
		boolean[][] visited = new boolean[m][n];

		for (int i = 0; i < m; i++) {
			for (int j = 0; j < n; j++) {
				if (mat[i][j] == 0) {
					queue.add(new int[] {i, j});
					visited[i][j] = true;
				}
			}
		}

		while (!queue.isEmpty()) {
			int[] cur = queue.poll();
			int curRow = cur[0], curCol = cur[1];
			for (int i = 0; i < 4; i++) {
				int newRow = curRow + dx[i];
				int newCol = curCol + dy[i];
				if (newRow >= 0 && newRow < m
					&& newCol >= 0 && newCol < n
					&& !visited[newRow][newCol]) {
					mat[newRow][newCol] = mat[curRow][curCol] + 1;
					visited[newRow][newCol] = true;
					queue.add(new int[] {newRow, newCol});
				}
			}
		}

		return mat;
	}
```