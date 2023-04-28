# 题目链接
https://leetcode.cn/problems/maximum-difference-between-node-and-ancestor/

# 解答过程
题目要求并不难理解，翻译一下，从root到每个叶子节点有一条path，这条path上最大值和最小值有一个差，现在要计算的是所有path中差最大的那一条对应的差是多少。

最直接粗暴的想法，那就遍历所有path嘛，每遍历完一条path，计算一下当前path中最大值和最小值的差，并维护最大差，当遍历完所有path后，即得到了最终答案。

如何遍历所有path呢？首先想到的是DFS，稍微麻烦的一点是如何在DFS的过程中维护当前的path，以及如何维护最大差。

```java
	public int maxAncestorDiff(TreeNode root) {
		List<TreeNode> path = new ArrayList<>();
		path.add(root);

		TreeNode maxDiffNodeMax = new TreeNode(0);
		TreeNode maxDiffNodeMin = new TreeNode(0);

		dfs(root, path, maxDiffNodeMax, maxDiffNodeMin);

		return Math.abs(maxDiffNodeMax.val - maxDiffNodeMin.val);
	}

	private void dfs(TreeNode cur, List<TreeNode> path, TreeNode maxDiffNodeMax, TreeNode maxDiffNodeMin) {
		if (cur.left == null
			&& cur.right == null) {
			TreeNode maxNode = path.get(0);
			TreeNode minNode = path.get(0);
			for (TreeNode node : path) {
				if (node.val > maxNode.val) {
					maxNode = node;
				}
				if (node.val < minNode.val) {
					minNode = node;
				}
			}

			if (Math.abs(maxNode.val - minNode.val) > Math.abs(maxDiffNodeMax.val - maxDiffNodeMin.val)) {
				maxDiffNodeMax.val = maxNode.val;
				maxDiffNodeMin.val = minNode.val;
			}

			return;
		}

		if (cur.left != null) {
			path.add(cur.left);
			dfs(cur.left, path, maxDiffNodeMax, maxDiffNodeMin);
			path.remove(path.size() - 1);
		}
		if (cur.right != null) {
			path.add(cur.right);
			dfs(cur.right, path, maxDiffNodeMax, maxDiffNodeMin);
			path.remove(path.size() - 1);
		}
	}
```

如果不使用递归的话，自己维护一个stack也可以，但是还需要维护一个节点是否已完全处理的状态，以及在节点出栈时要设置为已完全处理。

```java
	public int maxAncestorDiff(TreeNode root) {
		Set<TreeNode> done = new HashSet<>();

		Stack<TreeNode> path = new Stack<>();
		path.add(root);

		int maxDiff = 0;
		while (!path.isEmpty()) {
			TreeNode cur = path.peek();
			if (cur.left == null
				&& cur.right == null) {

				int pathMaxDiff = maxDiff(path);
				if (pathMaxDiff > maxDiff) {
					maxDiff = pathMaxDiff;
				}

				done.add(cur);
				path.pop();

				continue;
			}

			if (cur.left != null
				&& !done.contains(cur.left)) {
				path.push(cur.left);
				continue;
			}

			if (cur.right != null
				&& !done.contains(cur.right)) {
				path.push(cur.right);
				continue;
			}

			done.add(cur);
			path.pop();
		}

		return maxDiff;
	}

	private int maxDiff(Stack<TreeNode> path) {
		int max = path.get(0).val, min = path.get(0).val;

		for (TreeNode node : path) {
			if (node.val > max) {
				max = node.val;
			}
			if (node.val < min) {
				min = node.val;
			}
		}

		return max - min;
	}
```

上面两种写法都通过了，但是时间效率都很差，看了一眼官方题解，额，显然思路更高级，但是没完全理解，有时间再研究下。