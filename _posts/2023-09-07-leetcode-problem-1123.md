# 题目链接
https://leetcode.cn/problems/lowest-common-ancestor-of-deepest-leaves/

# 解答过程
这个题目乍一看有点迷糊，但结合示例稍加分析，可以明白题目的含义。一颗二叉树，每个节点都有一个depth，根节点depth为0，每往下每一层depth加1。这些节点中有一些是叶子节点，即没有孩子的节点。这些叶子节点可能有不同的depth，我们把depth最大的那些叶子节点挑出来，问题来了，这些被挑选出来的depth最大的叶子节点，它们的最小公共祖先是哪个节点？

解题思路嘛，第一直觉就是按上面的描述把这些depth最大的叶子节点找出来，这个处理还是比较简单的，层级遍历嘛，最后遍历到的那一层就是depth最大的叶子节点。那么如何确定这些叶子节点的最小公共祖先呢？想到的笨办法就是把每个节点的父节点信息维护起来，这样对于给定的一组节点，连续往上查找父节点，直到所有节点相等，那么我们就得到了最小公共祖先。

```java
	public TreeNode lcaDeepestLeaves(TreeNode root) {
		if (root == null) {
			return null;
		}

		Map<TreeNode, TreeNode> parents = new HashMap<>();
		parents.put(root, null);

		List<TreeNode> list = new ArrayList<>();

		Queue<TreeNode> queue = new LinkedList<>();
		queue.offer(root);
		while (!queue.isEmpty()) {
			list.clear();

			int size = queue.size();
			while (size-- > 0) {
				TreeNode node = queue.poll();

				list.add(node);

				if (node.left != null) {
					queue.offer(node.left);
					parents.put(node.left, node);
				}
				if (node.right != null) {
					queue.offer(node.right);
					parents.put(node.right, node);
				}
			}
		}

		if (list.size() == 1) {
			return list.get(0);
		}

		while (true) {
			if (allSame(list)) {
				return list.get(0);
			}

			list.replaceAll(parents::get);
		}
	}

	private boolean allSame(List<TreeNode> list) {
		TreeNode first = list.get(0);
		for (TreeNode node : list) {
			if (node != first) {
				return false;
			}
		}

		return true;
	}
```

代码提交以后是没有问题的，效率较差，但思路很好理解。看了一下官方题解，有点意思，确实思路更显高级，代码也更简洁。参考递归思路又写了下：

```java
	public TreeNode lcaDeepestLeaves(TreeNode root) {
		Object[] ret = recur(root);
		return (TreeNode) ret[1];
	}

	private Object[] recur(TreeNode node) {
		if (node == null) {
			return new Object[] {0, null};
		}

		Object[] left = recur(node.left);
		Object[] right = recur(node.right);

		int leftDepth = (int) left[0];
		TreeNode leftLca = (TreeNode) left[1];
		int rightDepth = (int) right[0];
		TreeNode rightLca = (TreeNode) right[1];
		if (leftDepth > rightDepth) {
			return new Object[] {leftDepth + 1, leftLca};
		} else if (leftDepth < rightDepth) {
			return new Object[] {rightDepth + 1, rightLca};
		} else {
			return new Object[] {leftDepth + 1, node};
		}
	}
```