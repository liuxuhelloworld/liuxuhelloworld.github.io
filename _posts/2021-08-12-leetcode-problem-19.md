# 题目链接
https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/

# 解答过程
这个题目是挺久以前写的了。先遍历计数，后遍历到待删除节点的前一节点后进行删除，我觉得这样就可以了。官方题解的栈解法和双指针解法，有一定启发性，但对这个题目而言，感觉完全没必要，纯属简单问题复杂化。

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