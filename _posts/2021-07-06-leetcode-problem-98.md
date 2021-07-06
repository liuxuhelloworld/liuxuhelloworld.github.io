# 题目链接
https://leetcode-cn.com/problems/validate-binary-search-tree/

# 解答过程
掌握好二叉树的遍历，这个题目应该还是比较容易解决的。使用递归的话关键在于抽象出节点加上下限的递归策略，使用非递归的话关键在于在中序遍历的基础上要记录下前序节点。

## 递归
```java
	public boolean isValidBST(TreeNode root) {
		return isValidBSTUsingRecur(root, null, null);
	}

	private boolean isValidBSTUsingRecur(TreeNode cur, Integer lower, Integer upper) {
		if (cur == null) {
			return true;
		}

		if (lower != null && cur.val <= lower) {
			return false;
		}

		if (upper != null && cur.val >= upper) {
			return false;
		}

		if (!isValidBSTUsingRecur(cur.left, lower, cur.val)) {
			return false;
		}

		if (!isValidBSTUsingRecur(cur.right, cur.val, upper)) {
			return false;
		}

		return true;
	}
```

## 非递归
```java
	public boolean isValidBSTV2(TreeNode root) {
		if (root == null) {
			return true;
		}

		Stack<TreeNode> stack = new Stack<>();
		TreeNode node = root;
		TreeNode prev = null;
		while (node != null
			|| !stack.isEmpty()) {
			while (node != null) {
				stack.push(node);
				node = node.left;
			}

			TreeNode cur = stack.pop();

			if (prev != null && cur.val <= prev.val) {
				return false;
			}

			prev = cur;

			if (cur.right != null) {
				node = cur.right;
			}
		}

		return true;
	}
```