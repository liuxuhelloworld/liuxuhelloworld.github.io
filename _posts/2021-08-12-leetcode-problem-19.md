# 题目链接
https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/

# 解答过程
这个题目我一开始的解法就是先遍历计数，然后遍历到待删除节点的前一节点后进行删除，我觉得这样简单清晰，没什么不好。但是考思考再三，还是写下双指针版本吧。所谓双指针，那就是一快一慢两个指针嘛，快的那个指针有明确的终点，即null，当快指针变为null时，慢指针应该在哪呢？从题目要求来说，慢指针应该在我们要删除的倒数第N个节点的前一个节点，这样就很方便能把倒数第N个节点删除掉。为了方便处理删除的恰好是第1个节点的情况，我们增加一个dummy节点。好了，先让快节点走吧，问题是走多远呢？如果你画个图，尝试分析几个实例，会发现脑袋更疼了。其实从一个最极端的case来入手更合适，假设现在要删除的就是第1个节点，快节点先走，应该走多远？要知道因为要删除的就是第1个节点，快节点先走以后，慢节点不需要走了，慢节点指向dummy节点就是我们期望的结果，也就是说，此时，需要快节点已经是null，也就是说，快节点需要走N+1步，之所以+1是因为我们是从dummy节点开始出发的。

```java
	public ListNode removeNthFromEnd(ListNode head, int n) {
		ListNode dummy = new ListNode(0);
		dummy.next = head;

		ListNode p = dummy, q = dummy;
		for (int i = 0; i <= n; i++) {
			p = p.next;
		}

		while (p != null) {
			p = p.next;
			q = q.next;
		}
		q.next = q.next.next;

		return dummy.next;
	}
```