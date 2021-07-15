# 题目链接
https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

# 解答过程
根据前序遍历和中序遍历的特点，可知前序遍历的第一个节点是根节点，中序遍历中根节点左边是左子树，右边是右子树，由此可以递归的构造下去：
```java
	public TreeNode buildTree(int[] preorder, int[] inorder) {
		return recurBuildTree(preorder, 0, preorder.length-1,
			inorder, 0, inorder.length-1);
	}

	private TreeNode recurBuildTree(int[] preorder, int preStart, int preEnd,
	                                int[] inorder, int inStart, int inEnd) {
		if (preStart > preEnd) {
			return null;
		}

		int val = preorder[preStart];
		TreeNode root = new TreeNode(val);

		if (preStart == preEnd) {
			return root;
		}

		int index = indexOf(inorder, inStart, inEnd, val);
		int leftTreeNodesNum = index - inStart;
		root.left = recurBuildTree(preorder, preStart + 1, preStart + leftTreeNodesNum,
			inorder, inStart, index - 1);
		root.right = recurBuildTree(preorder, preStart + leftTreeNodesNum + 1, preEnd,
			inorder, index + 1, inEnd);

		return root;
	}

	private int indexOf(int[] arr, int start, int end, int target) {
		for (int i = start; i <= end; i++) {
			if (arr[i] == target) {
				return i;
			}
		}

		return -1;
	}
```

我觉得这个思路还是比较好理解的，即便没接触过，也比较容易想到这样来解决问题。

看了一眼官方题解，基本差不多吧。但是第二种非递归的解法，呵呵，瞅了两眼就放弃了，脑仁疼，等什么时候有兴致了再研究下非递归的解法。