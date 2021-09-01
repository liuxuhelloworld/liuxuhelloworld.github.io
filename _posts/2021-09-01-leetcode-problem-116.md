# 题目链接
https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node/

# 解答过程
## BFS
这个题目其实就可以理解成二叉树层级遍历的变种，就是每一层从左到右维护成一个链表嘛，在层级遍历的基础上稍做修改即可：

```java
	public Node connect(Node root) {
		if (root == null) {
			return null;
		}

		Queue<Node> queue = new LinkedList<>();
		queue.add(root);

		while (!queue.isEmpty()) {
			int size = queue.size();

			Node dummy = new Node(0);
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