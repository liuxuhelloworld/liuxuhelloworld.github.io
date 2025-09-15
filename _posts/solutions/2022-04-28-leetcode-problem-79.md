# 题目链接
https://leetcode-cn.com/problems/word-search/

# 解答过程
这个题目是挺久以前写的了，当时应该也是参考官方题解写的。重新看代码逻辑，有些地方就不太能理解。对代码进行了一些”优化“后，重新提交，测试未通过。单步跑报错的测试用例，恍然大悟，而且一定程度上让我更好地理解了遍历和回溯。一直比较困惑的一个问题就是，DFS和backtrack区别在哪呢？之前写的一些DFS和backtrack的题目，从代码看，真得很像，甚至有的backtrack标签的题目代码里方法名就是dfs. 我觉得DFS和backtrack比较显著的区别应该是在布尔标记数组的处理上，都经常用到布尔标记数组来标记遍历或回溯的过程，区别在于DFS往往是一次性的、单纯的标记某个元素是否已遍历到，而backtrack往往需要在递归返回后重置布尔标记数组，以避免影响后续的回溯。听起来有点抽象，随便挑几个用到了布尔标记数组的DFS和backtrack题目的代码看一下，就很好理解了。

```java
	private int[] dx = new int[] {-1, 1, 0, 0};
	private int[] dy = new int[] {0, 0, -1, 1};

	public boolean exist(char[][] board, String word) {
		int m = board.length;
		int n = board[0].length;

		boolean[][] tag = new boolean[m][n];

		for (int i = 0; i < m; i++) {
			for (int j = 0; j < n; j++) {
				if (board[i][j] == word.charAt(0)) {
					resetTag(tag);
					tag[i][j] = true;
					boolean exists = backtrack(board, i, j, word, 0, tag);
					if (exists) {
						return true;
					}
				}
			}
		}

		return false;
	}

	private void resetTag(boolean[][] tag) {
		for (int i = 0; i < tag.length; i++) {
			for (int j = 0; j < tag[0].length; j++) {
				if (tag[i][j]) {
					tag[i][j] = false;
				}
			}
		}
	}

	private boolean backtrack(char[][] board, int curRow, int curCol, String word, int curIdx, boolean[][] tag) {
		if (board[curRow][curCol] != word.charAt(curIdx)) {
			return false;
		}

		if (curIdx == word.length()-1) {
			return true;
		}

		for (int i = 0; i < dx.length; i++) {
			int nextRow = curRow + dx[i];
			int nextCol = curCol + dy[i];
			if (isValidNextCell(nextRow, nextCol, word.charAt(curIdx+1), board, tag)) {
				tag[nextRow][nextCol] = true;
				boolean exists = backtrack(board, nextRow, nextCol, word,curIdx+1, tag);
				if (exists) {
					return true;
				}
				tag[nextRow][nextCol] = false;
			}
		}

		return false;
	}

	private boolean isValidNextCell(int nextRow, int nextCol, char nextCh, char[][] board, boolean[][] tag) {
		return isValidRow(nextRow, board.length)
			&& isValidCol(nextCol, board[0].length)
			&& tag[nextRow][nextCol] == false
			&& nextCh == board[nextRow][nextCol];
	}

	private boolean isValidRow(int row, int rowsNum) {
		return row >= 0 && row < rowsNum;
	}

	private boolean isValidCol(int col, int colsNum) {
		return col >= 0 && col < colsNum;
	}
```