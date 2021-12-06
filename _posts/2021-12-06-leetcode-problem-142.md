# 题目链接
https://leetcode-cn.com/problems/linked-list-cycle-ii/

# 解答过程
判断链表是否有环，有环则返回环的起始节点，否则返回null. 环的起始节点就是链表（传统意义上的）最后一个节点的下一个节点，所以记录每个节点的访问状态，当访问到某个访问状态是已访问的节点的时候，这个节点就是起始节点。

```java
	public ListNode detectCycle(ListNode head) {
		Set<ListNode> visited = new HashSet<>();

		for (ListNode p = head; p != null; p = p.next) {
			if (visited.contains(p)) {
				return p;
			}

			visited.add(p);
		}

		return null;
	}
```