# 题目链接
https://leetcode.cn/problems/check-if-matrix-is-x-matrix/

# 解答过程
这个题目比较简单了，稍微归纳一下，可以发现，对角线元素就是那些i == j或者i+j == n-1的元素，由此双循环遍历并判断即可。

```java
	public boolean checkXMatrix(int[][] grid) {
		int n = grid.length;

		for (int i = 0; i < n; i++) {
			for (int j = 0; j < n; j++) {
				if (i == j || i+j == n-1) {
					if (grid[i][j] == 0) {
						return false;
					}
				} else {
					if (grid[i][j] != 0) {
						return false;
					}
				}
			}
		}

		return true;
	}
```