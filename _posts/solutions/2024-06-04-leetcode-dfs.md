# 问题733 图像渲染 ★☆☆☆☆
https://leetcode-cn.com/problems/flood-fill/

这个题目和问题130、问题200、问题695是类似的，而且还更简单些，只需要从一个点开始做一次遍历，遍历的逻辑是一致的，不赘述。

```java
	int[] dx = new int[] {0, -1, 0, 1};
	int[] dy = new int[] {1, 0, -1, 0};

	public int[][] floodFill(int[][] image, int sr, int sc, int newColor) {
		dfs(image, sr, sc, image[sr][sc], newColor);
		return image;
	}

	private void dfs(int[][] image, int curRow, int curCol, int startColor, int newColor) {
		int curColor = image[curRow][curCol];
		if (curColor == newColor) {
			return;
		}

		if (curColor == startColor) {
			image[curRow][curCol] = newColor;
			for (int i = 0; i < 4; i++) {
				int newRow = curRow + dx[i];
				int newCol = curCol + dy[i];
				if (newRow >= 0
					&& newRow < image.length
					&& newCol >= 0
					&& newCol < image[0].length) {
					dfs(image, curRow + dx[i], curCol + dy[i], startColor, newColor);
				}
			}
		}
	}
```

# 问题797 所有可能的路径 ★★☆☆☆
https://leetcode-cn.com/problems/all-paths-from-source-to-target/

这个题目简单来说就是计算有向无环图的指定两个节点之间的所有路径。从DFS的角度来考虑，从某个节点开始，循环遍历所有它直接指向的节点，每次循环又是一个DFS，所谓深度优先。在遍历的过程中，需要维护一个结果list和一个当前path，如果当前path已经到了目的节点，那么当前path入结果list. 代码逻辑还是很好理解的，提交以后也通过了，不过效率较差。初步来看DFS过程中从src到dst有重复计算，加一层cache应该会好些，有时间再改下。

```java
	public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
		List<List<Integer>> result = new ArrayList<>();
		List<Integer> path = new ArrayList<>();

		path.add(0);
		dfs(graph, 0, graph.length - 1, result, path);

		return result;
	}

	private void dfs(int[][] graph, int src, int dst, List<List<Integer>> result, List<Integer> path) {
		if (src == dst) {
			result.add(path.stream().collect(Collectors.toList()));
			return;
		}

		for (int next : graph[src]) {
			path.add(next);
			dfs(graph, next, dst, result, path);
			path.remove(path.size() - 1);
		}
	}
```

# 问题200 岛屿数量 ★★☆☆☆
https://leetcode-cn.com/problems/number-of-islands/

从深度优先遍历的角度来考虑这个题目，思路就比较清晰了，以某一个节点为起始，按照一定顺序去深度优先遍历值为1的节点，并标注访问状态，直到没有能到达的值为1的节点结束，这样就等于把某一个连通的岛屿的所有节点全部访问到并标注了访问状态。上层做二维数组遍历，碰到值为1的节点并且尚未标注访问状态的，说明碰到了新岛屿，那么从这个节点开始做深度优先遍历。

这个题目和问题130很像，DFS的过程是一致的，相比问题130，问题200只需要计数，还更简单些。

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

# 问题130 被围绕的区域 ★★☆☆☆
https://leetcode-cn.com/problems/surrounded-regions/

这种二维数组按元素属性做连通的题目，可以说是比较典型的遍历类题目了。对于本题，从DFS的角度考虑，给定一个元素，如果元素值是X，跳过即可；如果元素值是O，那么开始遍历，目标是把和这个元素连通的所有元素都遍历到。比如，先往上，再往下，再往左，再往右，每个方向都遍历到不应该再继续遍历为止，所谓DFS。仅仅是遍历的话这样就可以了，但题目要求是把被X包围的O都改成X，怎么实现呢？这里一开始我就想得比较幼稚了，我的想法是把每次遍历到的元素都记录下来，遍历完再处理遍历到的元素集合，判断有没有某个元素是边界点，如果有，说明这些元素连接构成的区域未被X包围；如果没有，说明这些元素连接构成的区域被X包围了，那么把这些元素对应的值改为X。这个思路理解起来很容易，执行起来也是对的，只不过效率很差。稍作思考，我也想到了其实不必尝试遍历所有元素，仅对边界点并且元素值为O的做遍历，并且记录下遍历到的元素的位置，最后把除这些位置外其他元素都赋值X即可。这个稍微优化的思路，我没有修改代码提交验证，但想必效率会好很多。然后我就看了下官方题解，整体的思路是差不多的，但是官方题解更简洁，都不需要单独维护一个visited数组来记录被遍历到O，直接把被遍历到的原地改为A，最后再改回来...

```java
	private static int[] dx = {-1, 1, 0, 0};
	private static int[] dy = {0, 0, -1, 1};

	public void solve(char[][] board) {
		int rows = board.length;
		int cols = board[0].length;

		for (int j = 0; j < cols; j++) {
			if (board[0][j] == 'O') {
				dfs(board, 0, j);
			}
			if (board[rows-1][j] == 'O') {
				dfs(board, rows-1, j);
			}
		}

		for (int i = 0; i < rows; i++) {
			if (board[i][0] == 'O') {
				dfs(board, i, 0);
			}
			if (board[i][cols-1] == 'O') {
				dfs(board, i, cols-1);
			}
		}

		for (int i = 0; i < rows; i++) {
			for (int j = 0; j < cols; j++) {
				if (board[i][j] == 'A') {
					board[i][j] = 'O';
				} else if (board[i][j] == 'O') {
					board[i][j] = 'X';
				}
			}
		}
	}

	private void dfs(char[][] board, int i, int j) {
		board[i][j] = 'A';

		for (int k = 0; k < dx.length; k++) {
			int nextI = i + dx[k];
			int nextJ = j + dy[k];

			if (isValid(board, nextI, nextJ)
				&& board[nextI][nextJ] == 'O') {
				dfs(board, nextI, nextJ);
			}
		}
	}

	private boolean isValid(char[][] board, int i, int j) {
		int rows = board.length;
		int cols = board[0].length;

		if (i < 0 || i >= rows) {
			return false;
		}

		if (j < 0 || j >= cols) {
			return false;
		}

		return true;
	}
```

# 问题547 省份数量 ★★☆☆☆
https://leetcode-cn.com/problems/number-of-provinces/

这个题目和问题130-被围绕的区域、问题200-岛屿数量有一些相似之处，问题130和问题200这些题目里，每个元素是独立的个体，以自身的值为依据做连通。而这个题目里元素值表示的是这些个体之间是否连通，等于是给了一个图，要计算的是有多少连通的子图。对图算法敏感的话，应该第一时间就明白这是计算图的连通分量数目。从代码的角度看，Problem130、Problem200是以二维数组的每个元素为粒度做DFS，这个题目是以图的节点为粒度做DFS。

```java
	public int findCircleNum(int[][] isConnected) {
		int len = isConnected.length;

		boolean[] visited = new boolean[len];

		int num = 0;
		for (int i = 0; i < len; i++) {
			if (!visited[i]) {
				num++;
				dfs(isConnected, visited, i);
			}
		}

		return num;
	}

	private void dfs(int[][] isConnected, boolean[] visited, int i) {
		visited[i] = true;
		for (int j = 0; j < isConnected.length; j++) {
			if (isConnected[i][j] == 1
				&& !visited[j]) {
				dfs(isConnected, visited, j);
			}
		}
	}
```

# 问题695 岛屿的最大面积 ★★☆☆☆
https://leetcode-cn.com/problems/max-area-of-island/

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