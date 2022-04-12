# 题目链接
https://leetcode-cn.com/problems/subtree-of-another-tree/

# 解答过程
额，这个题目是看了官方题解以后写的。官方题解的思路比较容易理解，简单来说就是做两层DFS，第一层DFS用来遍历可能的每棵子树来比较，第二层DFS用来判断两颗树是否相等。而我一开始的思路是怎样在DFS过程中既遍历可能的每棵子树，又比较两颗树是否相等，即没有想到把比较两颗树抽象为独立的方法。

```java
	public boolean isSubtree(TreeNode root, TreeNode subRoot) {
		return dfs(root, subRoot);
	}

	private boolean dfs(TreeNode s, TreeNode t) {
		if (isSame(s, t)) {
			return true;
		}

		return s != null && (dfs(s.left, t) || dfs(s.right, t));
	}

	private boolean isSame(TreeNode s, TreeNode t) {
		if (s == null && t == null) {
			return true;
		} else if (s != null && t == null) {
			return false;
		} else if (s == null && t != null) {
			return false;
		} else {
			if (s.val != t.val) {
				return false;
			} else {
				return isSame(s.left, t.left) && isSame(s.right, t.right);
			}
		}
	}
```