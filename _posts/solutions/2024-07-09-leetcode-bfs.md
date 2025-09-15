# 问题733 图像渲染 ★☆☆☆☆
https://leetcode-cn.com/problems/flood-fill/

这个题目和问题130、问题200、问题695是类似的，而且还更简单些，只需要从一个点开始做一次遍历，遍历的逻辑是一致的，不赘述。

```java
	int[] dx = new int[] {0, -1, 0, 1};
	int[] dy = new int[] {1, 0, -1, 0};

	public int[][] floodFill(int[][] image, int sr, int sc, int newColor) {
		int targetColor = image[sr][sc];
		if (targetColor == newColor) {
			return image;
		}

		Queue<int[]> queue = new LinkedList<>();
		queue.add(new int[] {sr, sc});
		image[sr][sc] = newColor;

		while (!queue.isEmpty()) {
			int[] cur = queue.poll();
			for (int i = 0; i < 4; i++) {
				int newRow = cur[0] + dx[i];
				int newCol = cur[1] + dy[i];
				if (newRow >= 0 && newRow < image.length
					&& newCol >= 0 && newCol < image[0].length
					&& image[newRow][newCol] == targetColor) {
					queue.add(new int[] {newRow, newCol});
					image[newRow][newCol] = newColor;
				}
			}
		}

		return image;
	}
```

# 问题695 岛屿的最大面积 ★★☆☆☆
https://leetcode-cn.com/problems/max-area-of-island/

BFS的解法并不难理解。从某个点开始，上下左右四个方向，联通的点入队列，同时，为了避免重复，借助一个visited的布尔数组。

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

# 问题542 01矩阵 ★★☆☆☆
https://leetcode-cn.com/problems/01-matrix/

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

# 问题994 腐烂的橘子 ★★☆☆☆
https://leetcode-cn.com/problems/rotting-oranges/

这个题目其实和问题542很像，按分钟rotten就可以看作计算从rottened点到fresh点的距离，先把所有rottened的点看作一个整体入队列，然后每次循环处理时，把作为整体入队列的点当作一批来处理，直到队列为空。

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

# 问题1091 二进制矩阵中的最短路径 ★★☆☆☆
https://leetcode-cn.com/problems/shortest-path-in-binary-matrix/

这个题目和问题542、问题994很类似，稍有区别就是这个题目是可以斜着遍历，即可以以相邻点做遍历，而不限于相邻边。简单来说就是从源点开始按宽度做扩散，每次处理一批距离源点相同宽度的0点，判断这些0点中是否已经扩散到了终点，如果是，返回depth长度。如果不是，则把下一宽度能扩散到的0点入队列。如果遍历结束都没有扩散到终点，则没有clear path，返回-1。

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
		int depth = 0;

		while (!queue.isEmpty()) {
			depth++;

			int size = queue.size();
			while (size-- > 0) {
				int[] cur = queue.poll();
				int row = cur[0], col = cur[1];

				if (row == n-1 && col == n-1) {
					return depth;
				}

				for (int k = 0; k < dx.length; k++) {
					int newRow = row + dx[k];
					int newCol = col + dy[k];

					if (newRow < 0 || newRow >= n || newCol < 0 || newCol >= n) {
						continue;
					}

					if (grid[newRow][newCol] == 0
						&& !visited[newRow][newCol]) {
						queue.add(new int[] {newRow, newCol});
						visited[newRow][newCol] = true;
					}
				}
			}
		}

		return -1;
	}
```

# 问题1129 颜色交替的最短路径 ★★☆☆☆
https://leetcode.cn/problems/shortest-path-with-alternating-colors/

这个题目可是费了不少的时间和精力。从解题入手点来说，大的思路并不难，比较容易想到可以通过广度优先遍历（BFS）来解决这个问题。问题的关键在于每一层BFS具体做哪些操作？这可以从循环不变量来入手。我一开始的思路是每次进入循环时，当前维护着这样两个变量：shortestPathLength\[\]，记录的是从0节点到v节点的最短颜色交错路径长度；lastPathColor\[\]，记录的是从0节点到v节点的最短颜色交错路径的最后一条path的颜色，可能的取值有不可达、红色、蓝色、红蓝均可等四种情况。在当前遍历中，根据当前v节点的lastPathColor，决定是继续遍历redEdges或者blueEdges或者都遍历，同时更新遍历到的节点的shortestPath和lastPathColor。直到遍历结束，根据lastPathColor确定不可达的节点并更新shortestPathLength，然后返回。整个思路看起来还挺正确吧？处理了一些细节问题以后，最终的结果是超时，出现了死循环，因为可能出现环，对于已经遍历过的节点是否要二次遍历直觉上很难处理。

最终只得参考了下官方题解，看了一下官方题解，大的思路是类似的，但有一个地方启发了我，那就是分别维护shortestPathLengthWithRedLastPath\[\]和shortestPathLengthWithBlueLastPath\[\]，而不是妄图用一个lastPathColor\[\]来维护所有可能的状态。修改了一下代码，终于通过了，但是代码看起来有点复杂，不过相比官方题解的代码，我又觉得不如我自己写的好理解。

```java
	public int[] shortestAlternatingPaths(int n, int[][] redEdges, int[][] blueEdges) {
		int[] shortestPathLengthWithRedLastPath = new int[n];
		shortestPathLengthWithRedLastPath[0] = 0;
		for (int i = 1; i < n; i++) {
			shortestPathLengthWithRedLastPath[i] = -1;
		}

		int[] shortestPathLengthWithBlueLastPath = new int[n];
		shortestPathLengthWithBlueLastPath[0] = 0;
		for (int i = 1; i < n; i++) {
			shortestPathLengthWithBlueLastPath[i] = -1;
		}

		Queue<Integer> queue = new LinkedList<>();
		queue.offer(0);

		while (!queue.isEmpty()) {
			int v = queue.poll();

			if (shortestPathLengthWithBlueLastPath[v] != -1) {
				int length = shortestPathLengthWithBlueLastPath[v] + 1;
                for (int[] redEdge : redEdges) {
                    if (redEdge[0] == v) {
                        int next = redEdge[1];

                        if (shortestPathLengthWithRedLastPath[next] == -1) {
                            shortestPathLengthWithRedLastPath[next] = length;
                            queue.add(next);
                        } else {
                            if (length < shortestPathLengthWithRedLastPath[next]) {
                                shortestPathLengthWithRedLastPath[next] = length;
                            }
                        }
                    }
                }
			}

			if (shortestPathLengthWithRedLastPath[v] != -1) {
				int length = shortestPathLengthWithRedLastPath[v] + 1;
                for (int[] blueEdge : blueEdges) {
                    if (blueEdge[0] == v) {
                        int next = blueEdge[1];

                        if (shortestPathLengthWithBlueLastPath[next] == -1) {
                            shortestPathLengthWithBlueLastPath[next] = length;
                            queue.add(next);
                        } else {
                            if (length < shortestPathLengthWithBlueLastPath[next]) {
                                shortestPathLengthWithBlueLastPath[next] = length;
                            }
                        }
                    }
                }
			}
		}

		int[] shortestPathLength = new int[n];
		for (int i = 0; i < n; i++) {
			int withRedLastPath = shortestPathLengthWithRedLastPath[i];
			int withBlueLastPath = shortestPathLengthWithBlueLastPath[i];

			if (withRedLastPath == -1
				&& withBlueLastPath == -1) {
				shortestPathLength[i] = -1;
			} else if (withRedLastPath == -1) {
				shortestPathLength[i] = withBlueLastPath;
			} else if (withBlueLastPath == -1) {
				shortestPathLength[i] = withRedLastPath;
			} else {
				shortestPathLength[i] = Math.min(withRedLastPath, withBlueLastPath);
			}
		}

		return shortestPathLength;
	}
```