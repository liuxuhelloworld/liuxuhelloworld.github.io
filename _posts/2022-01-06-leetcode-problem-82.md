# 题目链接
https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/

# 解答过程
这个题目和Problem83相似，但这里要求删除所有重复值节点，比Problem83处理起来就麻烦一些。每次循环处理时，要先计数当前节点值，如果是重复值节点，直接跳过即可；如果不是重复值节点，追加到维护的新链表的最后一个节点上。

```java
	public ListNode deleteDuplicates(ListNode head) {
		ListNode dummy = new ListNode(-1);
		ListNode slow, fast;

		slow = dummy;
		fast = head;
		while (fast != null) {
			int target = fast.val;
			int cnt = 1;

			while (fast.next != null) {
				if (fast.next.val == target) {
					fast = fast.next;
					cnt++;
				} else {
					break;
				}
			}

			if (cnt == 1) {
				slow.next = fast;
				slow = slow.next;
			} else {
				slow.next = null;
			}

			fast = fast.next;
		}

		return dummy.next;
	}
```

瞅了眼官方题解，思路其实是类似的，官方题解的代码更简洁但也更难懂一些，我认为我的代码更好理解。