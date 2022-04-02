# 题目链接
https://leetcode-cn.com/problems/01-matrix/

# 解答过程
这个题目是看了官方题解以后写的，自己一开始还是想用DFS，很快就陷入了双向依赖的死循环；也有考虑动态规划，但想的是怎么设计一个统一的动态规划方程，而实际只能是四个方向分别做动态规划。其实从题目要求来看，计算每个节点离最近的0点的距离，BFS应该是更明显的思路，按宽度扩展嘛。对于BFS，关键点在于入队列的时机，即按宽度从小到大入队列。对于本题，就是按距离最近0点的距离入队列。即先入队列的是所有0点，然后入队列的是所有距离0点为1的点，再然后入队列的是所有距离0点为2的点......，以此类推，这样就计算出了每个点到最近的0点的距离。

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