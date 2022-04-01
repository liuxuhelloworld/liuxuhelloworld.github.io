# 题目链接
https://leetcode-cn.com/problems/surrounded-regions/

# 解答过程
## DFS
这种二维数组按元素属性做连通的题目，可以说是比较典型的遍历类题目了。对于本题，从DFS的角度考虑，给定一个元素，如果元素值是X，跳过即可；如果元素值是O，那么开始遍历，目标是把和这个元素连通的所有元素都遍历到。比如，先往上，再往下，再往左，再往右，每个方向都遍历到不应该再继续遍历为止，所谓DFS。仅仅是遍历的话这样就可以了，但题目要求是把被X包围的O都改成X，怎么实现呢？这里一开始我就想得比较幼稚了，我的想法是把每次遍历到的元素都记录下来，遍历完再处理遍历到的元素集合，判断有没有某个元素是边界点，如果有，说明这些元素连接构成的区域未被X包围；如果没有，说明这些元素连接构成的区域被X包围了，那么把这些元素对应的值改为X。这个思路理解起来很容易，执行起来也是对的，只不过效率很差。稍作思考，我也想到了其实不必尝试遍历所有元素，仅对边界点并且元素值为O的做遍历，并且记录下遍历到的元素的位置，最后把除这些位置外其他元素都赋值X即可。这个稍微优化的思路，我没有修改代码提交验证，但想必效率会好很多。然后我就看了下官方题解，整体的思路是差不多的，但是官方题解更简洁，都不需要单独维护一个visited数组来记录被遍历到O，直接把被遍历到的原地改为A。

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