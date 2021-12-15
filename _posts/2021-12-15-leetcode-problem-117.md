# 题目链接
https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node-ii/

# 解答过程
使用层序遍历比较容易解决这个问题。写完以后才发现这个题目和Problem116几乎就是一样的，按照层序遍历处理的话代码都是通用的。比较了下和Problem116的代码，发现基本是一样的，但还是稍有差别，You cannot step into the same river twice.

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

