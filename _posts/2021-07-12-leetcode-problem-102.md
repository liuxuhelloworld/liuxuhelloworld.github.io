# 题目链接
https://leetcode-cn.com/problems/binary-tree-level-order-traversal/

# 解答过程
这个题就是BFS稍微变通一下，感觉中等难度有点给高了。

```java
	public List<List<Integer>> levelOrder(TreeNode root) {
		if (root == null) {
			return Collections.emptyList();
		}

		List<List<Integer>> ret = new ArrayList<>();

		Queue<TreeNode> queue = new LinkedList<>();
		queue.add(root);

		while (!queue.isEmpty()) {
			int size = queue.size();

			List<Integer> curLevel = new ArrayList<>(size);
			while (size-- > 0) {
				TreeNode cur = queue.poll();
				curLevel.add(cur.val);

				if (cur.left != null) {
					queue.add(cur.left);
				}
				if (cur.right != null) {
					queue.add(cur.right);
				}
			}

			ret.add(curLevel);
		}

		return ret;
	}
```