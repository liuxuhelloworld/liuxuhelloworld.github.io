# 题目链接

https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/

# 解法1
最直接的思路，先计数链表长度，然后定位到要删除节点的上一个节点。

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

# 解法2

添加一个dummy节点，先让两个节点p和q拉开n个间隔的距离，然后p和q同时前进，p为空时即q定位到了要删除节点的上一节点。

我觉得这种方法并没有降低访问链接节点的次数，反而增加了思路的复杂度，尤其是不如第1种解法方便处理入参n的校验。

```java
	public ListNode removeNthFromEndV2(ListNode head, int n) {
		if (head == null || n <= 0) {
			return head;
		}

		ListNode dummy = new ListNode(-1);
		dummy.next = head;

		ListNode p = dummy, q = dummy;
		for (int i = 0; i < n+1; i++) {
			if (p != null) {
				p = p.next;
			} else {
				return head;
			}
		}

		while (p != null) {
			p = p.next;
			q = q.next;
		}
		q.next = q.next.next;

		return dummy.next;
	}
```



