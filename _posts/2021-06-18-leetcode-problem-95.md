# 题目链接
https://leetcode-cn.com/problems/unique-binary-search-trees-ii/

# 解答过程
一开始面对这个题目，有点头疼，problem 96就简单多了，probem 95这个涉及树的具体形态，就感觉比较麻烦。

着手点还是动态规划，在S(n-1)的基础上怎么得到S(n)呢？简单来说，把一个val为n的新节点依次加到S(n-1)的每颗树上就可以了，对吧？那么给定S(n-1)的一颗树，怎么把新节点n加到这颗树上，并且保持二叉搜索树的结构呢？因为新节点n最大，所以得从右子树入手。原树从根节点沿右子树一直走下去直到null节点，用新节点依次占据这条路径上每个节点的位置，被占据的节点作为新节点n的左子树即可。从直觉上，我觉得这样操作就是对的，当然，数学证明我是做不来的。

```java
	public List<TreeNode> generateTrees(int n) {
		assert n >= 1 && n <= 8;

		List<TreeNode> ret = new ArrayList<>();
		ret.add(new TreeNode(1));

		for (int i = 2; i <= n; i++) {
			List<TreeNode> newRet = new ArrayList<>();

			for (TreeNode node : ret) {
				newRet.addAll(addNewNode(node, i));
			}

			ret = newRet;
		}

		return ret;
	}

	private List<TreeNode> addNewNode(TreeNode node, int val) {
		List<TreeNode> ret = new ArrayList<>();

		int depth = getDepthOfRightestNode(node);

		for (int i = 0; i <= depth; i++) {
			TreeNode copy = copy(node);
			TreeNode prev = null, cur = copy;

			int j = 0;
			while (j++ < i) {
				prev = cur;
				cur = cur.right;
			}

			TreeNode newVal = new TreeNode(val);
			newVal.left = cur;
			if (prev == null) {
				ret.add(newVal);
			} else {
				prev.right = newVal;
				ret.add(copy);
			}
		}

		return ret;
	}

	private int getDepthOfRightestNode(TreeNode node) {
		int depth = 0;
		while (node != null) {
			depth++;
			node = node.right;
		}

		return depth;
	}

	private TreeNode copy(TreeNode node) {
		if (node == null) {
			return null;
		}

		TreeNode copy = new TreeNode(node.val);
		copy.left = copy(node.left);
		copy.right = copy(node.right);

		return copy;
	}
```

提交以后确实是没有问题的。这个题前后花了应该有一个半小时，如果真的在面试中碰到，我估计可能写不出来。又看了下官方题解，用回溯法应该更容易想出来，抽时间再整一把。