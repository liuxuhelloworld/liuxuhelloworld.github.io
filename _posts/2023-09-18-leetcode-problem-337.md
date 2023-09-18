# 题目链接
https://leetcode.cn/problems/house-robber-iii/

# 解答过程
这个题目有点意思，一开始我没搞清楚题目的意思，认为可行的解法是：给定一个节点，获取左子树的最大rob数，获取右子树的最大rob数，如果左子树和右子树最大rob数之和大于当前节点的val，那么返回它们的和作为当前节点的子树的最大rob数，否则，返回val作为当前节点的最大rob数。递归很好写嘛，但是，以题目中的示例去验证就会发现问题，这种解法当然满足题目中节点不相邻的限制，但不是最优解。

那么应该如何处理呢？给定一个节点，我们有两种选择，一种是选定这个节点，一种是不选定这个节点，我们需要的其实是这样的信息：在选定这个节点时，这颗子树的最大rob数是多少？在不选定这个节点时，这颗子树的最大rob数是多少？即\[selected, unselected\]。如果有了这样的信息，那么给定一个节点，我们可以先分别计算它的左子树和右子树，得到\[leftSelected, leftUnselected\]和\[rightSelected, rightUnselected\]，那么对于当前节点，selected = val + leftUnselected + rightUnselected，unselected = max(leftSelected, leftUnselected)+(rightSelected, rightUnselected)，以此递归可得根节点的\[selected, unselected\]，它们中的较大值即为最优解。

```java
	public int rob(TreeNode root) {
		int[] rob = dfs(root);
		return Math.max(rob[0], rob[1]);
	}

	private int[] dfs(TreeNode node) {
		if (node == null) {
			return new int[] {0, 0};
		}

		int[] left = dfs(node.left);
		int[] right = dfs(node.right);
		int val = node.val;

		int leftSelected = left[0];
		int leftUnselected = left[1];
		int rightSelected = right[0];
		int rightUnselected = right[1];
		int nodeSelected = val + leftUnselected + rightUnselected;
		int nodeUnselected = Math.max(leftSelected, leftUnselected) + Math.max(rightSelected, rightUnselected);

		return new int[] {nodeSelected, nodeUnselected};
	}
```