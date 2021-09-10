# 题目链接
https://leetcode-cn.com/problems/reverse-linked-list/

# 解答过程
反转单链表，很基础的题了，没啥好说的。

```java
	public ListNode reverseList(ListNode head) {
		ListNode dummy = new ListNode();

		ListNode p = head;
		while (p != null) {
			ListNode tmp = p.next;
			p.next = dummy.next;
			dummy.next = p;
			p = tmp;
		}

		return dummy.next;
	}
```