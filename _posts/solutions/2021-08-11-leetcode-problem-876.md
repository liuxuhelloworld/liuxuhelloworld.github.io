# 题目链接
https://leetcode-cn.com/problems/middle-of-the-linked-list/

# 解答过程
这个题目看起来是很简单的，最笨的方法就是遍历两次嘛，第一次计数总长度，第二次定位中间节点。那能不能一次遍历就完成呢？我们用cur节点遍历链表的过程中，维护一个mid节点，表示到cur为止的子链表的中间节点，当cur遍历到链表最后时，mid就是整个链表的中间节点。进一步的，按照归纳法可以总结得出：当cur节点的下标是偶数时，mid节点不动；当cur节点的下标是奇数时，mid向前一步：

```java
	public ListNode middleNode(ListNode head) {
		ListNode cur = head, mid = head;
		int curIndex = 0;

		while (cur != null) {
			if (curIndex % 2 == 1) {
				mid = mid.next;
			}

			cur = cur.next;
			curIndex++;
		}

		return mid;
	}
```