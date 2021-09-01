# 题目链接
https://leetcode-cn.com/problems/merge-two-binary-trees/

# 解答过程
## DFS
这个题目我觉得不需要从DFS或BFS的角度考虑，就从递归的角度考虑就很好理解了。merge两个二叉树，那么显然可以理解为：merge它们的左子树，merge它们的右子树，merge根节点，并把merge左右子树的结果作为新的左右子树：

```java
	public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
		if (root1 == null && root2 == null) {
			return null;
		} else if (root1 != null && root2 == null) {
			return root1;
		} else if (root1 == null && root2 != null) {
			return root2;
		} else {
			TreeNode left = mergeTrees(root1.left, root2.left);
			TreeNode right = mergeTrees(root1.right, root2.right);
			root1.val = root1.val + root2.val;
			root1.left = left;
			root1.right = right;

			return root1;
		}
	}
```

当然，这个递归的背后就是DFS，但我还是觉得单纯按递归来考虑更清晰简洁。

## BFS
简单瞅了瞅官方题解的BFS，感觉纯属没病找病，等我哪天闲得蛋疼了再写个BFS版本吧。