# 题目链接
https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/

# 解答过程
这个题和problem 105就很相似了，思路是一样的，不赘述。

```java
	public TreeNode buildTree(int[] inorder, int[] postorder) {
		return recurBuildTree(inorder, 0, inorder.length - 1,
			postorder, 0, postorder.length - 1);
	}

	private TreeNode recurBuildTree(int[] inorder, int inStart, int inEnd,
	                                int[] postorder, int postStart, int postEnd) {
		if (postStart > postEnd) {
			return null;
		}

		int val = postorder[postEnd];
		TreeNode root = new TreeNode(val);

		if (postStart == postEnd) {
			return root;
		}

		int index = indexOf(inorder, inStart, inEnd, val);
		int leftTreeNodesNum = index - inStart;
		root.left = recurBuildTree(inorder, inStart, index - 1,
			postorder, postStart, postStart + leftTreeNodesNum - 1);
		root.right = recurBuildTree(inorder, index + 1, inEnd,
			postorder, postStart + leftTreeNodesNum, postEnd - 1);

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

看了眼官方题解的递归写法，感觉还没我写的看起来简洁:dog: