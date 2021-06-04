# 题目链接
https://leetcode-cn.com/problems/reverse-linked-list-ii/

# 解答过程
这个题给我的第一感觉就是简单，应该很好写的样子，但是第一次尝试确没能在半小时内写出来，原因我觉得是在没有一个清晰的思路前就着急下笔，导致写的很乱，也算是给自己上了一课。后面静下心来，先确定好一个思路，然后再动手：
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

我用的方法没有什么复杂的算法，就是简单直接的遍历，解答的正确性也非常依赖入参的合法性，代码看起来比较丑......