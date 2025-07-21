# 题目链接
https://leetcode-cn.com/problems/reverse-linked-list-ii/

# 解答过程
这个题目其实难度不大，主要考察链表操作的编码能力。我们只需要遍历链表，在遍历到目标区间时维护一个逆序的子链表，并且把逆序子链表和前后节点连接起来即可。在遍历过程中，通过使用dummy节点可以很方便地完成这些操作。

```java
	public ListNode reverseBetween(ListNode head, int left, int right) {
		assert head != null;
		assert left >= 1 && right >= 1 && left <= right;

		ListNode dummy = new ListNode(-1);
		dummy.next = head;

		ListNode p = dummy;
		for (int i = 1; i < left; i++) {
			p = p.next;
		}

		ListNode prevLeft = p;
		ListNode reversedTail = p.next;

		ListNode reversedDummy = new ListNode(-1);
		ListNode cur = p.next;
		ListNode next = null;
		for (int i = 0; i < right-left+1; i++) {
			next = cur.next;
			cur.next = reversedDummy.next;
			reversedDummy.next = cur;
			cur = next;
		}

		prevLeft.next = reversedDummy.next;
		reversedTail.next = next;

		return dummy.next;
	}
```