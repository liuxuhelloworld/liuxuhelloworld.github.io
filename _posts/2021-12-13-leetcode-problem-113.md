# 题目链接
https://leetcode-cn.com/problems/path-sum-ii/

# 解答过程
## DFS
我觉得这个题目的解决思路还是比较容易想到的，就是对树做遍历，动态维护当前路径和目标值。

```java
	public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
		List<List<Integer>> result = new ArrayList<>();
		List<Integer> cur = new ArrayList<>();

		dfs(result, cur, root, targetSum);

		return result;
	}

	private void dfs(List<List<Integer>> result, List<Integer> cur, TreeNode node, int target) {
		if (node == null) {
			return;
		}

		if (isLeafNode(node)) {
			if (node.val == target) {
				cur.add(node.val);
				result.add(clone(cur));
				cur.remove(cur.size() - 1);
			}
			return;
		}


		if (node.left != null) {
			cur.add(node.val);
			dfs(result, cur, node.left, target - node.val);
			cur.remove(cur.size() - 1);
		}

		if (node.right != null) {
			cur.add(node.val);
			dfs(result, cur, node.right, target - node.val);
			cur.remove(cur.size() - 1);
		}
	}

	private boolean isLeafNode(TreeNode node) {
		return node != null
			&& node.left == null
			&& node.right == null;
	}

	private List<Integer> clone(List<Integer> list) {
		return list.stream()
			.collect(Collectors.toList());
	}
```

看了下官方题解，思路是差不多的，不过我的代码稍冗余一些，但我自己觉得更好理解。