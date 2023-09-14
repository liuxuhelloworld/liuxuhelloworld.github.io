# 题目链接
https://leetcode.cn/problems/queens-that-can-attack-the-king/

# 解答过程
结合示例，题目并不难理解，一个国际象棋棋盘，有一个位置是king，其他一些位置是queen，沿着横向、纵向、对角三条线，离king最近的queen可以直接攻击king，注意必须是最近的，不能隔着queen去攻击king，现在需要找出这些能直接攻击king的queen。

解题思路呢？我觉得很直接嘛，从king的位置开始，沿着向右、向右上、向上、向左上、向左、向左下、向下、向右下8个方向，挨个方向遍历就好了，碰到第一个queen就是能直接攻击king的queue，那么这个方向的遍历就可以退出了。但是题目的入参不是一个二维数组，而是queen和king的位置，没关系，我们自己构造一个二维数组就好了。

```java
	public List<List<Integer>> queensAttacktheKing(int[][] queens, int[] king) {
		int[][] board = new int[8][8];

		for (int[] queen : queens) {
			int queenRow = queen[0];
			int queenCol = queen[1];
			board[queenRow][queenCol] = 1;
		}

		int kingRow = king[0];
		int kingCol = king[1];
		board[kingRow][kingCol] = 2;

		List<List<Integer>> attacks = new ArrayList<>();

		// go right
		for (int j = kingCol + 1; j < 8; j++) {
			if (board[kingRow][j] == 1) {
				attacks.add(Arrays.asList(kingRow, j));
				break;
			}
		}

		// go up-right
		for (int i = kingRow - 1, j = kingCol + 1; i >= 0 && j < 8; i--, j++) {
			if (board[i][j] == 1) {
				attacks.add(Arrays.asList(i, j));
				break;
			}
		}

		// go up
		for (int i = kingRow - 1; i >= 0; i--) {
			if (board[i][kingCol] == 1) {
				attacks.add(Arrays.asList(i, kingCol));
				break;
			}
		}

		// go up-left
		for (int i = kingRow - 1, j = kingCol - 1; i >= 0 && j >= 0; i--, j--) {
			if (board[i][j] == 1) {
				attacks.add(Arrays.asList(i, j));
				break;
			}
		}

		// go left
		for (int j = kingCol - 1; j >= 0; j--) {
			if (board[kingRow][j] == 1) {
				attacks.add(Arrays.asList(kingRow, j));
				break;
			}
		}

		// go down-left
		for (int i = kingRow + 1, j = kingCol - 1; i < 8 && j >= 0; i++, j--) {
			if (board[i][j] == 1) {
				attacks.add(Arrays.asList(i, j));
				break;
			}
		}

		// go down
		for (int i = kingRow + 1; i < 8; i++) {
			if (board[i][kingCol] == 1) {
				attacks.add(Arrays.asList(i, kingCol));
				break;
			}
		}

		// go down-right
		for (int i = kingRow + 1, j = kingCol + 1; i < 8 && j < 8; i++, j++) {
			if (board[i][j] == 1) {
				attacks.add(Arrays.asList(i, j));
				break;
			}
		}

		return attacks;
	}
```

看了一眼官方题解，我觉得还不如我这种方法更清晰。另外，这个题目我觉得真算不上中等难度......