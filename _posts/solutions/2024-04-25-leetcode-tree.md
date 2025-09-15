# 问题617 合并二叉树 ★☆☆☆☆
https://leetcode-cn.com/problems/merge-two-binary-trees/

这个题目从递归的角度出发并不难想到解题思路，merge两个二叉树可以处理为：merge它们的左子树，merge它们的右子树，merge根节点，并把merge左右子树的结果作为新的左右子树。

如果不使用递归呢？不使用递归的话，对两颗二叉树做层序遍历，然后一层一层的merge，处理起来也不复杂，需要注意对于空节点，需要按null入队列，这样才能确认需要merge的节点之间的对应位置。

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

# 问题102 二叉树的层序遍历 ★★☆☆☆
https://leetcode-cn.com/problems/binary-tree-level-order-traversal/

这个题目就是层序遍历，不赘述。

```java
	public List<List<Integer>> levelOrder(TreeNode root) {
		if (root == null) {
			return Collections.emptyList();
		}

		List<List<Integer>> ret = new ArrayList<>();

		Queue<TreeNode> queue = new LinkedList<>();
		queue.add(root);

		while (!queue.isEmpty()) {
			int size = queue.size();

			List<Integer> curLevel = new ArrayList<>(size);
			while (size-- > 0) {
				TreeNode cur = queue.poll();
				curLevel.add(cur.val);

				if (cur.left != null) {
					queue.add(cur.left);
				}
				if (cur.right != null) {
					queue.add(cur.right);
				}
			}

			ret.add(curLevel);
		}

		return ret;
	}
```

# 问题107 二叉树的层序遍历 ★★☆☆☆
https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/

额， 这个就在层序遍历的结果上加一个reverse就可以了，我觉得这道题比较多余。

```java
	public List<List<Integer>> levelOrderBottom(TreeNode root) {
		if (root == null) {
			return Collections.emptyList();
		}

		List<List<Integer>> ret = new ArrayList<>();

		Queue<TreeNode> queue = new LinkedList<>();
		queue.add(root);

		while (!queue.isEmpty()) {
			int size = queue.size();

			List<Integer> curLevel = new ArrayList<>(size);
			while (size-- > 0) {
				TreeNode cur = queue.poll();
				curLevel.add(cur.val);

				if (cur.left != null) {
					queue.add(cur.left);
				}
				if (cur.right != null) {
					queue.add(cur.right);
				}
			}

			ret.add(curLevel);
		}

		Collections.reverse(ret);

		return ret;
	}
```

# 问题103 二叉树的锯齿形层序遍历 ★★☆☆☆
https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/

这个题目就是把层序遍历稍微改动下，如果从入队列时的顺序入手考虑，感觉就很麻烦，其实就还是按层序遍历入队出队，根据当前层级，加一个reverse就好了。

```java
	public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
		if (root == null) {
			return Collections.emptyList();
		}

		List<List<Integer>> ret = new ArrayList<>();

		Queue<TreeNode> queue = new LinkedList<>();
		queue.add(root);

		int curLevel = 0;

		while (!queue.isEmpty()) {
			int size = queue.size();

			List<Integer> curLevelList = new ArrayList<>(size);
			while (size-- > 0) {
				TreeNode cur = queue.poll();
				curLevelList.add(cur.val);

				if (cur.left != null) {
					queue.add(cur.left);
				}
				if (cur.right != null) {
					queue.add(cur.right);
				}
			}

			if (curLevel % 2 != 0) {
				Collections.reverse(curLevelList);
			}
			ret.add(curLevelList);
			curLevel++;
		}

		return ret;
	}
```

# 问题116 填充每个节点的下一个右侧节点指针 ★★☆☆☆
https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/

这个题目其实就是二叉树层序遍历的变种，就是每一层从左到右维护成一个链表嘛，在层序遍历的基础上稍做修改即可。

```java
	public Node connect(Node root) {
		if (root == null) {
			return null;
		}

		Queue<Node> queue = new LinkedList<>();
		queue.add(root);

		Node dummy = new Node(-1);

		while (!queue.isEmpty()) {
			int size = queue.size();

			Node last = dummy;
			for (int i = 0; i < size; i++) {
				Node cur = queue.poll();
				last.next = cur;
				last = cur;

				if (cur.left != null) {
					queue.offer(cur.left);
				}
				if (cur.right != null) {
					queue.offer(cur.right);
				}
			}
		}

		return root;
	}
```

# 问题117 填充每个节点的下一个右侧节点指针 ★★☆☆☆
https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node-ii/

这个题目和Problem116几乎就是一样的，按照层序遍历处理的话代码都是通用的，不赘述。

# 问题572 另一棵树的子树 ★★☆☆☆
https://leetcode-cn.com/problems/subtree-of-another-tree/

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

# 问题113 路径总和 ★★☆☆☆
https://leetcode-cn.com/problems/path-sum-ii/

这个题目需要做的是遍历二叉树的每一条path，可以称为path遍历，而path遍历其实就是前序遍历，只不过我们需要在遍历过程中维护当前的path

```java
	public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
		List<List<Integer>> result = new ArrayList<>();
		List<TreeNode> path = new ArrayList<>();

		dfs(result, path, root, targetSum);

		return result;
	}

	private void dfs(List<List<Integer>> result, List<TreeNode> path, TreeNode node, int target) {
		if (node == null) {
			return;
		}

		path.add(node);

		if (node.left == null
			&& node.right == null) {
			if (node.val == target) {
				result.add(path.stream().map(e -> e.val).toList());
			}
		}

		if (node.left != null) {
			dfs(result, path, node.left, target - node.val);
		}

		if (node.right != null) {
			dfs(result, path, node.right, target - node.val);
		}

		path.remove(path.size() - 1);
	}
```

完全可以抽象出path遍历的代码如下：
```java
	private static void paths(TreeNode node, List<TreeNode> path) {
		if (node == null) {
			return;
		}

		path.add(node);

		if (node.left == null
			&& node.right == null) {
			pathVisit(path);
		}

		if (node.left != null) {
			paths(node.left, path);
		}

		if (node.right != null) {
			paths(node.right, path);
		}

		path.remove(path.size() - 1);
	}

	private static void pathVisit(List<TreeNode> path) {
		List<String> vals = path.stream()
			.map(node -> node.val)
			.map(String::valueOf)
			.toList();
		System.out.println(String.join(",", vals));
	}
```

# 问题1026 节点与其祖先之间的最大差值 ★★☆☆☆
https://leetcode.cn/problems/maximum-difference-between-node-and-ancestor/

题目要求并不难理解，翻译一下，从root到每个叶子节点有一条path，这条path上最大值和最小值有一个差，现在要计算的是所有path中差最大的那一条对应的差是多少。

最直接粗暴的想法就是做路径遍历嘛，遍历所有path，每遍历完一条path，计算一下当前path中最大值和最小值的差，并维护最大差，当遍历完所有path后，即得到了最终答案。

```java
	public int maxAncestorDiff(TreeNode root) {
		List<TreeNode> path = new ArrayList<>();
		path.add(root);

		return dfs(root, path, 0);
	}

	private int dfs(TreeNode cur, List<TreeNode> path, int maxDiff) {
		if (cur.left == null
			&& cur.right == null) {
			int pathMaxDiff = pathMaxDiff(path);
            return Math.max(pathMaxDiff, maxDiff);
		}

		if (cur.left != null) {
			path.add(cur.left);
			maxDiff = dfs(cur.left, path, maxDiff);
			path.remove(path.size() - 1);
		}
		if (cur.right != null) {
			path.add(cur.right);
			maxDiff = dfs(cur.right, path, maxDiff);
			path.remove(path.size() - 1);
		}

		return maxDiff;
	}

	private int pathMaxDiff(List<TreeNode> path) {
		int max = path.get(0).val, min = path.get(0).val;
		for (TreeNode node : path) {
			if (node.val > max) {
				max = node.val;
			}
			if (node.val < min) {
				min = node.val;
			}
		}

		return max - min;
	}
```

路径遍历的解法我觉得在思路和实现上都很好理解，但是效率较差，确实，每次遍历到叶子节点，还要遍历当前path来计算最大差。那么有没有办法能优化呢？官方题解的解法我看了个云里雾里，但是有一点启发了我，那就是能不能把当前path省略，直接动态维护的就是当前path的节点最大值和最小值，尝试了一下，OK的，时间效率也好了很多。另外，相比官方题解，我觉得下面的代码更好理解。

```java
	public int maxAncestorDiff(TreeNode root) {
		return dfs(root, root.val, root.val, 0);
	}

	private int dfs(TreeNode cur, int max, int min, int maxDiff) {
		if (cur == null) {
			return maxDiff;
		}

		max = Math.max(cur.val, max);
		min = Math.min(cur.val, min);
		maxDiff = Math.max(maxDiff, max - min);

		if (cur.left == null
			&& cur.right == null) {
			return maxDiff;
		}

		if (cur.left != null) {
			maxDiff = dfs(cur.left, max, min, maxDiff);
		}
		if (cur.right != null) {
			maxDiff = dfs(cur.right, max, min, maxDiff);
		}

		return maxDiff;
	}
```

# 问题98 验证二叉搜索树
https://leetcode-cn.com/problems/validate-binary-search-tree/

如何判断一颗二叉树是不是BST呢？根据BST的定义，本质上就是判断这颗二叉树的中序遍历是不是递增的。如果是递增的，那说明这是一颗BST，如果不是递增的，则说明这不是一颗有效的BST。既然如此，那就中序遍历就可以了，笨一点的办法是将中序遍历的结果存储到一个list或stack中，无论是递归还是非递归，然后判断list或stack中元素的顺序是否符合要求。那么能不能在遍历过程中动态检查元素大小关系，从而判定是否有效的BST呢？也可以的，只不过稍微麻烦点。至于官方题解里的递归写法，确实简洁，抽时间再写一下。

使用递归：
```java
	public boolean isValidBST(TreeNode root) {
		Stack<TreeNode> stack = new Stack<>();
		return inOrder(root, stack);
	}

	private boolean inOrder(TreeNode node, Stack<TreeNode> stack) {
		if (node == null) {
			return true;
		}

		if (node.left != null) {
			boolean ret = inOrder(node.left, stack);
			if (!ret) {
				return false;
			}
		}

		if (!stack.isEmpty()) {
			TreeNode last = stack.peek();
			if (node.val <= last.val) {
				return false;
			}
		}

		stack.add(node);

		if (node.right != null) {
			boolean ret = inOrder(node.right, stack);
			if (!ret) {
				return false;
			}
		}

		return true;
	}
```

不使用递归：
```java
	public boolean isValidBST(TreeNode root) {
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

# 问题99 恢复二叉搜索树
https://leetcode-cn.com/problems/recover-binary-search-tree/

根据题目要求，一颗二叉搜索树，有两个节点的位置被对调了，现在需要恢复到对调前的状态，使它变为有效的二叉搜索树。首先这里不需要纠结于操作树指针，只需要定位到两个被对调的节点，然后交换它们的值就可以了。那么如何定位到两个被对调的节点呢？还得从二叉搜索树的定义入手。我们知道，一个二叉搜索树的中序遍历是一个递增序列，对调两个节点，等效于在一个递增序列中对调两个元素，根据前后元素的大小关系，我们是可以很方便地定位到这两个元素的。

这个题目和问题98是比较相似的，都是以二叉搜索树的中序遍历来做文章。

至于题目中空间复杂度O(1)的要求，瞅了眼下官方题解，额，有时间再研究下这个Morris中序遍历。

```java
	public void recoverTree(TreeNode root) {
		List<TreeNode> inOrderList = new ArrayList<>();

		inOrderTraverse(root, inOrderList);

		TreeNode swappedLarge = null, swappedSmall = null;
		for (int i = 0; i < inOrderList.size() - 1; i++) {
			TreeNode cur = inOrderList.get(i);
			TreeNode next = inOrderList.get(i+1);
			if (cur.val > next.val) {
				swappedLarge = cur;
				break;
			}
		}
		for (int i = inOrderList.size() - 1; i > 0; i--) {
			TreeNode cur = inOrderList.get(i);
			TreeNode prev = inOrderList.get(i-1);
			if (cur.val < prev.val) {
				swappedSmall = cur;
				break;
			}
		}

		assert swappedLarge != null;
		assert swappedSmall != null;

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

# 问题96 不同的二叉搜索树
https://leetcode-cn.com/problems/unique-binary-search-trees/

这个题目如果纠结于具体的二叉搜索树形态就想错方向了，关键点是将问题抽象，从整棵树的角度来考虑。我们定义包含从1到n这n个节点的可能的二叉搜索树的个数为S(n)。这些二叉搜索树可能以从1到n的任一个作为根节点，如果以1作为根节点，那么对应的可能的二叉搜索树有多少个呢？因为1是最小值，那么左子树显然是空，右子树包含n-1个节点，所以以1作为根节点，可能的二叉搜索树的个数是1\*S(n-1)。同理，以2作为根节点，可能的二叉搜索树的个数是S(1)\*S(n-2)。推而广之，以k作为根节点，可能的二叉搜索树的个数是S(k-1)\*S(n-k)。所以，包含n个节点的可能的二叉搜索树S(n)=1\*S(n-1)+S(1)\*S(n-2)+...+S(k-1)\*S(n-k)+...+S(n-1)\*1。

```java
	public int numTrees(int n) {
		int[] num = new int[n+1];
		num[0] = 1;

		for (int i = 1; i <= n; i++) {
			for (int j = 0; j <= i-1; j++) {
				num[i] += num[j] * num[i-1-j];
			}
		}

		return num[n];
	}
```

# 题目95 不同的二叉搜索树
https://leetcode-cn.com/problems/unique-binary-search-trees-ii/

这个题目相比问题96就要复杂一些，问题96中我们不关心二叉搜索树的具体形态，只关心可能的数量，而问题95这里就必须要处理二叉搜索树具体的形态了。

这个题目给人感觉很复杂的样子，对于这种没有太好入手点的题目，先考虑最简单的情况，由小到大的思考解题方法。对于这个题目而言，如果n=1，那么很简单一个单节点的树就可以了。如果n=2呢？也很简单，要么把节点2作为节点1的右子树，要么把节点1作为节点2的左子树。进一步的，如果n=3呢？更进一步的，如果我们有了n=k的解，如何得到n=k+1的解呢？简单来说，就是把一个值为k+1的新节点依次加到n=k的解包含的每颗树上就可以了，对吧？那么给定n=k的解的一颗树，怎么把新节点k+1加到这颗树上，并且保持二叉搜索树的结构呢？因为新节点k+1最大，所以得从右子树入手。原树从根节点沿右子树一直走下去直到null节点，用新节点依次占据这条路径上每个节点的位置，被占据的节点作为新节点k+1的左子树即可。从直觉上，我觉得这样操作就是对的，虽然数学证明我是做不来的。

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

这种方法是没有问题的，效率也不错，就是代码乍一看不够简洁，那么有没有更简洁的解法呢？参考官方题解我又下了下面一版使用递归的解法。递归解法也不难理解，从递归函数的形式就能理解解题思路。

```java
	public List<TreeNode> generateTrees(int n) {
		assert n >= 1 && n <= 8;

		return generateTrees(1, n);
	}

	private List<TreeNode> generateTrees(int x, int y) {
		if (x > y) {
			List<TreeNode> trees = new ArrayList<>();
			trees.add(null);
			return trees;
		}

		List<TreeNode> trees = new ArrayList<>();
		for (int i = x; i <= y; i++) {
			List<TreeNode> leftTrees = generateTrees(x, i-1);
			List<TreeNode> rightTrees = generateTrees(i+1, y);

			for (TreeNode left : leftTrees) {
				for (TreeNode right : rightTrees) {
					TreeNode root = new TreeNode(i);
					root.left = left;
					root.right = right;
					trees.add(root);
				}
			}
		}

		return trees;
	}
```

在调试递归解法的过程中，发现有很多重复计算，稍作分析就想到，这不是正合适用动态规划么。于是又写了下面一版。有点奇怪的是，动态规划这一版时间效率最差。另外，因为Java不支持范型数组，所以代码看起来比较丑。

```java
	public List<TreeNode> generateTrees(int n) {
		assert n >= 1 && n <= 8;

		List[][] dp = new List[n+1][n+1]; // dp[i..j] represents the BSTs for i..j
		for (int i = 1; i <= n; i++) {
			dp[i][i] = Arrays.asList(new TreeNode(i));
		}

		for (int step = 1; step <= n-1; step++) {
			for (int i = 1; i <= n - step; i++) {
				List<TreeNode> trees = new ArrayList<>();

				for (int k = i; k <= i+step; k++) {
					TreeNode root = new TreeNode(k);

					List leftTrees = new ArrayList();
					leftTrees.add(null);
					if (k-1 >= i) {
						leftTrees = dp[i][k-1];
					}

					List rightTrees = new ArrayList();
					rightTrees.add(null);
					if (k+1 <= i+step) {
						rightTrees = dp[k+1][i+step];
					}

					for (Object left : leftTrees) {
						for (Object right : rightTrees) {
							root.left = (TreeNode) left;
							root.right = (TreeNode) right;
							trees.add(copy(root));
						}
					}
				}

				dp[i][i+step] = trees;
			}
		}

		return dp[1][n];
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

# 问题105 从前序与中序遍历序列构造二叉树
https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

根据前序遍历和中序遍历的特点，可知前序遍历的第一个节点是根节点，中序遍历中根节点左边是左子树，右边是右子树，由此可以递归的构造下去。我觉得这个思路还是比较好理解的，即便没接触过，也比较容易想到这样来解决问题。至于官方题解中非递归的解法，呵呵，瞅了两眼就觉得脑袋疼，等什么时候有兴致了再研究下非递归的解法。

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

# 问题106 从中序与后序遍历序列构造二叉树
https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/

这个题目和问题105如出一辙，思路是一样的，不赘述。

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

# 问题449 序列化和反序列化二叉搜索树 ★★☆☆☆
https://leetcode.cn/problems/serialize-and-deserialize-bst/

题目要求很好理解了，简单来说，给定一颗二叉查找树，写两个方法，一个方法可以把二叉查找树序列化为字符串，另一个方法可以把序列化的字符串反序列化为同样结构的二叉查找树。

我是从反序列化方法入手的，先搞清楚如何从一个字符串能构建出一颗二叉查找树，这里面隐含着我们需要怎样的一个字符串，这个字符串需要让我们比较方便的重建二叉查找树。那么我们需要怎样的一个字符串呢？这就要从二叉查找树这个特性入手了，因为单纯的一个字符串直觉上很难包含树结构信息。这个地方稍作思考，可以发现，如果我们有二叉查找树先序遍历的结果，那么就可以比较方便的构造，逻辑如下：第一个元素显然是根节点，剩下的元素可以一分为二，左边的是左子树，右边的是右子树，而继续构造左子树或右子树的过程也是如此，利用递归很容易实现这个逻辑。OK，通过先序遍历的结果可以重构二叉查找树了，那么如果通过字符串得到先序遍历的结果呢？或者反过来说，如果通过先序遍历的结果得到一个字符串呢？这种转换显然是可以相互的。这个地方再稍作思考，比较简单直接的方法就是把先序遍历的结果以数组的形式存储，然后把这个数组转换为字符串。一个数组就是连续的字节序列，转换为字符串的简单方法就是每四位比特为一组解释为十六进制字符串。

代码提交以后是没有问题的，但是感觉有点复杂。看了一下官方题解，思路是差不多的，不过这个直接把遍历结果toString的处理还是有点，额，序列化的结果未免看着太不专业了吧......

```java
	public String serialize(TreeNode root) {
		if (root == null) {
			return null;
		}

		List<Integer> inOrderList = new ArrayList<>();
		inOrderTraverse(root, inOrderList);

		StringBuilder builder = new StringBuilder();
		for (int i = 0; i < inOrderList.size(); i++) {
			short cur = inOrderList.get(i).shortValue();

			int val = (cur & 0b1111000000000000) >>> 12;
			char ch = map(val);
			builder.append(ch);

			val = (cur & 0b0000111100000000) >>> 8;
			ch = map(val);
			builder.append(ch);

			val = (cur & 0b0000000011110000) >>> 4;
			ch = map(val);
			builder.append(ch);

			val = cur & 0b0000000000001111;
			ch = map(val);
			builder.append(ch);
		}

		return builder.toString();
	}

	private void inOrderTraverse(TreeNode node, List<Integer> inOrderList) {
		if (node == null) {
			return;
		}

		inOrderList.add(node.val);
		inOrderTraverse(node.left, inOrderList);
		inOrderTraverse(node.right, inOrderList);
	}

	private char map(int val) {
		char ch = 0;
		switch (val) {
			case 0:
				ch = '0';
				break;
			case 1:
				ch = '1';
				break;
			case 2:
				ch = '2';
				break;
			case 3:
				ch = '3';
				break;
			case 4:
				ch = '4';
				break;
			case 5:
				ch = '5';
				break;
			case 6:
				ch = '6';
				break;
			case 7:
				ch = '7';
				break;
			case 8:
				ch = '8';
				break;
			case 9:
				ch = '9';
				break;
			case 10:
				ch = 'A';
				break;
			case 11:
				ch = 'B';
				break;
			case 12:
				ch = 'C';
				break;
			case 13:
				ch = 'D';
				break;
			case 14:
				ch = 'E';
				break;
			case 15:
				ch = 'F';
				break;
			default:
				break;
		}

		return ch;
	}

	public TreeNode deserialize(String data) {
		if (data == null
			|| data.isEmpty()) {
			return null;
		}

		char[] chars = data.toCharArray();
		int length = chars.length;

		short[] inOrderArray = new short[length / 4];

		for (int i = 0; i < inOrderArray.length; i++) {
			char ch1 = chars[i*4];
			char ch2 = chars[i*4 + 1];
			char ch3 = chars[i*4 + 2];
			char ch4 = chars[i*4 + 3];

			int cur = 0;

			int val = map(ch1);
			cur = cur | (val << 12);

			val = map(ch2);
			cur = cur | (val << 8);

			val = map(ch3);
			cur = cur | (val << 4);

			val = map(ch4);
			cur = cur | val;

			inOrderArray[i] = (short) cur;
		}

		return rebuild(inOrderArray, 0, inOrderArray.length - 1);
	}

	private int map(char ch) {
		int val = 0;
		switch (ch) {
			case '0':
				val = 0;
				break;
			case '1':
				val = 1;
				break;
			case '2':
				val = 2;
				break;
			case '3':
				val = 3;
				break;
			case '4':
				val = 4;
				break;
			case '5':
				val = 5;
				break;
			case '6':
				val = 6;
				break;
			case '7':
				val = 7;
				break;
			case '8':
				val = 8;
				break;
			case '9':
				val = 9;
				break;
			case 'A':
				val = 10;
				break;
			case 'B':
				val = 11;
				break;
			case 'C':
				val = 12;
				break;
			case 'D':
				val = 13;
				break;
			case 'E':
				val = 14;
				break;
			case 'F':
				val = 15;
				break;
			default:
				break;
		}

		return val;
	}

	private TreeNode rebuild(short[] inOrderArray, int start, int end) {
		if (start > end) {
			return null;
		}

		short val = inOrderArray[start];
		TreeNode node = new TreeNode(val);

		int lessThan = -1;
		int greaterThan = -1;
		for (int i = start + 1; i <= end; i++) {
			if (inOrderArray[i] < val) {
				lessThan = i;
			} else if (inOrderArray[i] > val) {
				greaterThan = i;
				break;
			}
		}

		if (lessThan != -1) {
			node.left = rebuild(inOrderArray, start + 1, lessThan);
		}
		if (greaterThan != -1) {
			node.right = rebuild(inOrderArray, greaterThan, end);
		}

		return node;
	}
```

# 问题1110 删点成林 ★★☆☆☆
https://leetcode.cn/problems/delete-nodes-and-return-forest/

这个题目并不难理解，但是刚入手时并没有什么好思路。首先想到的是递归，但是并不方便将大问题转化为子问题，也便作罢。还是从示例问题入手，尝试通过简单的例子寻找解题思路。以示例问题为例，当删除某个value的节点A时，需要做哪些事情呢？
1. 需要将以节点A为父节点的节点的父节点置为null
2. 需要将节点A的父节点对应到节点A的左节点或右节点置为null
3. 将节点A删除

由以上结论，直觉上比较容易想到我们需要将每个节点的父节点维护起来，方便我们快速将某个节点的父节点置为null，或者由某个节点找到它的父节点进而修改父节点的左节点或右节点的值。根据题目限制，节点数是有限的，节点的值也是有限的，所以通过数组就可以很方便维护每个节点的父节点。

完成删除操作以后，剩下的forests的根节点就是父节点为null的那些。但是考虑到可能有不存在的节点的父节点值也是null，我们再维护一个self数组，这个数组记录的是每个节点本身，用于最后过滤掉不存在或已删除的节点。

额，坦白讲，代码看起来是有一点凌乱有一点丑的，提交以后可以通过，但是效率也较差。

看了下官方题解，好吧，DFS的代码确实简洁很多，但我总感觉不容易想出来......

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

# 问题1123 最深叶子节点的最近公共祖先 ★★☆☆☆
https://leetcode.cn/problems/lowest-common-ancestor-of-deepest-leaves/

题目本身并不难理解，一颗二叉树，每个节点都有一个depth，根节点depth为0，每往下每一层depth加1。这些节点中有一些是叶子节点，即没有孩子的节点。这些叶子节点可能有不同的depth，我们把depth最大的那些叶子节点挑出来。问题来了，这些被挑选出来的depth最大的叶子节点，它们的最小公共祖先是哪个节点？

我一开始的解题思路是层序遍历，并维护起来每个节点的父节点，根据层序遍历找到的depth最大的节点集合，通过维护的父节点信息去定位最小公共祖先。这种方法可以解决这个问题，不过代码看起来繁杂一些，效率也较差。

看了一下官方题解，有点意思，确实思路更显高级，代码也更简洁。关键点在于想明白这样一点：根据题目要求，最小公共祖先出现在depth较大的那颗子树上，如果左右子树的depth相等，那么当前的根节点就是最小公共祖先。

```java
	public TreeNode lcaDeepestLeaves(TreeNode root) {
		List<Object> ret = recur(root);
		return (TreeNode) ret.get(1);
	}

	private List<Object> recur(TreeNode node) {
		if (node == null) {
			return Arrays.asList(0, null);
		}

		List<Object> left = recur(node.left);
		List<Object> right = recur(node.right);

		int leftDepth = (int) left.get(0);
		TreeNode leftLca = (TreeNode) left.get(1);
		int rightDepth = (int) right.get(0);
		TreeNode rightLca = (TreeNode) right.get(1);
		if (leftDepth > rightDepth) {
			return Arrays.asList(leftDepth + 1, leftLca);
		} else if (leftDepth < rightDepth) {
			return Arrays.asList(rightDepth + 1, rightLca);
		} else {
			return Arrays.asList(leftDepth + 1, node);
		}
	}
```