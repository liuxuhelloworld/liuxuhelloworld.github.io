# 题目链接
https://leetcode-cn.com/problems/flood-fill/

# 解答过程
额，这个题目和Problem130、Problem200、Problem695是类似的，而且还更简单些，只需要从一个点开始做一次遍历，遍历的逻辑是一致的，不赘述。

## DFS
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

## BFS
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