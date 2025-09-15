# 题目链接
https://leetcode.cn/problems/merge-in-between-linked-lists/

# 解答过程
这个题目难度定义为中等属实有点高了，就是基本的遍历链表，考虑到入参的限制也较严，更是容易处理。

```java
	public ListNode mergeInBetween(ListNode list1, int a, int b, ListNode list2) {
		ListNode head = list2, tail = list2;
		while (tail.next != null) {
			tail = tail.next;
		}

		ListNode p = list1, q;
		int idx = 0;
		while (idx != a - 1) {
			p = p.next;
			idx++;
		}
		q = p.next;
		idx++;
		while (idx != b) {
			q = q.next;
			idx++;
		}

		p.next = head;
		tail.next = q.next;
		q.next = null;

		return list1;
	}
```