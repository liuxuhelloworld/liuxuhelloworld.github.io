# 题目链接
https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/

# 解答过程
额， 这个就在层级遍历的结果上加一个reverse就可以了，我觉得这道题比较多余

```java
	public List<List<Integer>> levelOrderBottom(TreeNode root) {
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

		Collections.reverse(ret);

		return ret;
	}
```