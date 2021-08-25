# 题目链接
https://leetcode-cn.com/problems/flood-fill/

# 解答过程
## DFS
对DFS的理解和掌握始终差点意思，这个题是看了官方题解后写的，看了官方题解又觉得很自然的处理逻辑，但总是对DFS代码缺少直觉。

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