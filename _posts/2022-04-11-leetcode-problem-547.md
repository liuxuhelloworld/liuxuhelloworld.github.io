# 题目链接
https://leetcode-cn.com/problems/number-of-provinces/

# 解答过程
## DFS
这个题目我一开始写错了，把问题想简单了，一开始我的理解是计数二维数组中互相连通的值为1的区域有几个，也就是和Problem130、Problem200类似。提交以后当然没通过，看了下报错的测试用例才明白这个题目的不同之处。Problem130、Problem200这些题目里，每个元素是独立的个体，以自身的值为依据做连通。而这个题目里，需要理解成以行或列为独立的个体，元素值表示的是这些个体之间是否连通。稍微再思考一下，才明白这个题目等于是给了一个图，要计算的是有多少连通的子图。对图算法敏感的话，应该第一时间就明白就是计算图的连通分量数目，可惜我对图算法的掌握太差......。从代码的角度看，Problem130、Problem200是以二维数组的每个元素为粒度做DFS，这个题目是以图的节点为粒度做DFS.

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