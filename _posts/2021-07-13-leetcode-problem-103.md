# 题目链接
https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/

# 解答过程
这个题目就是把层级遍历稍微改动下，如果从入队列时的顺序入手考虑，感觉就很麻烦，其实就还是按层级遍历入队出队，根据当前层级，加一个reverse就好了。

```java
	public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
		if (root == null) {
			return Collections.emptyList();
		}

		List<List<Integer>> ret = new ArrayList<>();

		Queue<TreeNode> queue = new LinkedList<>();
		queue.add(root);

		int curLevel = 0;

		while (!queue.isEmpty()) {
			int size = queue.size();

			List<Integer> curLevelList = new ArrayList<>(size);
			while (size-- > 0) {
				TreeNode cur = queue.poll();
				curLevelList.add(cur.val);

				if (cur.left != null) {
					queue.add(cur.left);
				}
				if (cur.right != null) {
					queue.add(cur.right);
				}
			}

			if (curLevel % 2 != 0) {
				Collections.reverse(curLevelList);
			}
			ret.add(curLevelList);
			curLevel++;
		}

		return ret;
	}
```

感觉这个题目也不应该算中等难度，怀疑自己是不是最近刷题刷的有点飘了[手动狗头]。