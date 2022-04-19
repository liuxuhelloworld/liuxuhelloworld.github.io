# 题目链接
https://leetcode-cn.com/problems/merge-two-sorted-lists/

# 解答过程
合并两个有序链表，和mergeSort里合并两个有序数组类似，双指针循环就可以了。瞅了一眼官方题解，递归的写法也很简洁。

```java
	public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
		if (l1 == null && l2 == null) {
			return null;
		}

		ListNode dummy = new ListNode();
		ListNode last = dummy;

		ListNode p = l1, q = l2;
		while (p != null && q != null) {
			if (p.val <= q.val) {
				last.next = p;
				p = p.next;
			} else {
				last.next = q;
				q = q.next;
			}
			last = last.next;
		}

		if (p != null) {
			last.next = p;
		}

		if (q != null) {
			last.next = q;
		}

		return dummy.next;
	}
```