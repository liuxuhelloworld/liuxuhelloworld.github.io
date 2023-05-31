# 题目链接
https://leetcode.cn/problems/delete-nodes-and-return-forest/

# 解答过程
这个题目并不难理解，但是刚入手时并没有什么好思路。首先想到的是递归，但是并不方便将大问题转化为子问题，也便作罢。还是从示例问题入手，尝试通过简单的例子寻找解题思路。以示例问题为例，当删除某个value的节点A时，需要做哪些事情呢？
1. 需要将以节点A为父节点的节点的父节点置为null
2. 需要将节点A的父节点对应到节点A的左节点或右节点置为null
3. 将节点A删除

由以上结论，直觉上比较容易想到我们需要将每个节点的父节点维护起来，方便我们快速将某个节点的父节点置为null，或者由某个节点找到它的父节点进而修改父节点的左节点或右节点的值。根据题目限制，节点数是有限的，节点的值也是有限的，所以通过数组就可以很方便维护每个节点的父节点。

完成删除操作以后，剩下的forests的根节点就是父节点为null的那些。但是考虑到可能有不存在的节点的父节点值也是null，我们再维护一个self数组，这个数组记录的是每个节点本身，用于最后过滤掉不存在或已删除的节点。

```java
	public List<TreeNode> delNodes(TreeNode root, int[] to_delete) {
		TreeNode[] parents = new TreeNode[1001]; // parent[i] represents the value i node's parent
		TreeNode[] self = new TreeNode[1001]; // self[i] represents the value i node self

		preOrder(root, parents, self);

		for (int i = 0; i < to_delete.length; i++) {
			int toDeleteVal = to_delete[i];

			for (int j = 0; j < parents.length; j++) {
				if (parents[j] != null
					&& parents[j].val == toDeleteVal) {
					parents[j] = null;
				}
			}

			TreeNode parent = parents[toDeleteVal];
			if (parent != null) {
				boolean isLeftChild = parent.left != null && parent.left.val == toDeleteVal;
				if (isLeftChild) {
					parent.left = null;
				} else {
					parent.right = null;
				}
			}

			self[toDeleteVal] = null;
		}

		List<TreeNode> forest = new ArrayList<>();
		for (int i = 1; i < parents.length; i++) {
			TreeNode parent = parents[i];
			if (parent == null
				&& self[i] != null) {
				forest.add(self[i]);
			}
		}

		return forest;
	}

	private void preOrder(TreeNode node, TreeNode[] parents, TreeNode[] self) {
		if (node == null) {
			return;
		}

		self[node.val] = node;

		if (node.left != null) {
			TreeNode left = node.left;
			parents[left.val] = node;
			preOrder(left, parents, self);
		}

		if (node.right != null) {
			TreeNode right = node.right;
			parents[right.val] = node;
			preOrder(right, parents, self);
		}
	}
```

额，坦白讲，代码看起来是有一点凌乱有一点丑的，提交以后可以通过，但是效率也较差。

看了下官方题解，好吧，还是我对递归的掌握不行，这个题目确实应当着手于深度优先遍历。