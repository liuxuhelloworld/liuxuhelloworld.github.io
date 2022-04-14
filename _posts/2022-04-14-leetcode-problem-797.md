# 题目链接
https://leetcode-cn.com/problems/all-paths-from-source-to-target/

# 解答过程
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