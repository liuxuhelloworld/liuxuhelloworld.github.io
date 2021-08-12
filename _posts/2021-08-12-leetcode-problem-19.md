# 题目链接
https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/

# 解答过程
我觉得这个题就简单的计算总长度，然后定位并删除就可以了：
```java
	public ListNode removeNthFromEnd(ListNode head, int n) {
		if (head == null || n <= 0) {
			return head;
		}

		int size = size(head);
		if (size == n) {
			head = head.next;
		} else if (size > n) {
			ListNode p = head;
			int i = 1;
			while (i++ < size-n) {
				p = p.next;
			}
			p.next = p.next.next;
		}

		return head;
	}

	private int size(ListNode node) {
		int size = 0;
		while (node != null) {
			size++;
			node = node.next;
		}

		return size;
	}
```

看了下官方题解的双指针解法，这里用所谓双指针在效率上没差吧，反而不好理解，容易写错。至于官方题解还有个用栈的解法，呵呵，你开心就好。