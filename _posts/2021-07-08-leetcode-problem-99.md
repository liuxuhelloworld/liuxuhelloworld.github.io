# 题目链接
https://leetcode-cn.com/problems/recover-binary-search-tree/

# 解答过程
一开始有点懵，没有什么思路，因为可能是任意两个节点的位置对调了。这种情况，我就会从最笨的方法去想，正常中序遍历的结果是一个递增序列，既然只有两个节点对调了，那么同样是做中序遍历，然后根据被破坏的大小关系，就可以确定被对调的节点了。

## 递归
```java
	public void recoverTree(TreeNode root) {
		List<TreeNode> inOrderList = new ArrayList<>();

		inOrderTraverse(root, inOrderList);

		TreeNode swappedLarge = null, swappedSmall = null;
		TreeNode prev = null;
		for (TreeNode cur : inOrderList) {
			if (prev != null && prev.val > cur.val) {
				swappedSmall = cur;
				if (swappedLarge == null) {
					swappedLarge = prev;
				}
			}
			prev = cur;
		}

		int tmp = swappedLarge.val;
		swappedLarge.val = swappedSmall.val;
		swappedSmall.val = tmp;
	}

	private void inOrderTraverse(TreeNode node, List<TreeNode> inOrderList) {
		if (node == null) {
			return;
		}

		inOrderTraverse(node.left, inOrderList);
		inOrderList.add(node);
		inOrderTraverse(node.right, inOrderList);
	}
```

## 非递归
```java
	public void recoverTreeV2(TreeNode root) {
		assert root != null;

		Stack<TreeNode> stack = new Stack<>();
		TreeNode node = root;
		TreeNode prev = null;
		TreeNode swappedLarge = null, swappedSmall = null;
		while (node != null
			|| !stack.isEmpty()) {
			while (node != null) {
				stack.push(node);
				node = node.left;
			}

			TreeNode cur = stack.pop();
			if (prev != null && prev.val > cur.val) {
				swappedSmall = cur;
				if (swappedLarge == null) {
					swappedLarge = prev;
				}
			}
			prev = cur;

			if (cur.right != null) {
				node = cur.right;
			}
		}

		int tmp = swappedLarge.val;
		swappedLarge.val = swappedSmall.val;
		swappedSmall.val = tmp;
	}
```

两种写法都还是比较清晰的，但都不满足题目中空间复杂度O(1)的要求。

瞅了眼下官方题解，额，有时间再研究下这个Morris中序遍历吧。