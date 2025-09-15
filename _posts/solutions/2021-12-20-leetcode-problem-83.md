# 题目链接
https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/

# 解答过程
这个题目比较简单，我们维护一个last，这个last表示新链表的最后一个节点，很容易进行循环不变量的论证：
1. last初始化为head，此时新链表只有last一个节点，满足题目要求；
2. 每次循环处理cur时，判断cur是否与last的val一样，如果不一样，说明cur可以追加到last后面，并更新last；如果一样，说明是重复节点，跳过即可；
3. 循环结束时，last为新链表的最后一个节点，并且新链表无重复节点，再置last.next为null即可。

```java
	public ListNode deleteDuplicates(ListNode head) {
		if (head == null) {
			return null;
		}

		ListNode last = head;
		for (ListNode cur = head.next; cur != null; cur = cur.next) {
			if (cur.val != last.val) {
				last.next = cur;
				last = cur;
			}
		}
		last.next = null;

		return head;
	}
```

这个题目和Problem82相似，但Problem82是去掉所有重复元素，处理起来就比Problem83麻烦一些了。Problem83碰到新val，先吃进来就可以。Problem82碰到新val，直接吃进来就不合适，因为有可能是重复元素，所以需要计数后再决定是不是能吃进来。