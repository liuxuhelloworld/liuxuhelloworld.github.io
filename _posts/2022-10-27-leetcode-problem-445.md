# 题目链接
https://leetcode.cn/problems/add-two-numbers-ii/

# 解答过程
这个题目和Problem2相似，区别在于Problem2里链表是倒序表示的整数，Problem445里链表是正序表示的整数。由于整数加法是从最低位到最高位处理，所以Problem2相对简单，只需要注意进位问题就可以了。那么比较直接的想法，就是把Problem445的链表做反转，相加以后再反转得到最终结果。然而题目中也指出，有没有解法可以不使用反转。这个地方我思考了一下，想到的是递归解法。从代码来看，其实还比较好理解，只是进位的问题处理起来有点难看，使用了一个成员变量来存储递归过程中每次计算后的进位值。

看了一下官方题解，额，果然是我把问题想复杂了。使用栈确实是更合乎逻辑、代码更简洁的解法。

```java
	private int carry = 0;

	public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
		int size1 = size(l1);
		int size2 = size(l2);

		ListNode ret = recur(l1, size1, l2, size2);
		if (carry != 0) {
			ListNode head = new ListNode(1);
			head.next = ret;
			return head;
		}

		return ret;
	}

	private int size(ListNode list) {
		int size = 0;
		while (list != null) {
			size++;
			list = list.next;
		}

		return size;
	}

	private ListNode recur(ListNode l1, int size1, ListNode l2, int size2) {
		if (size1 == 0) {
			return l2;
		}
		if (size2 == 0) {
			return l1;
		}

		ListNode next;

		int val1 = 0, val2 = 0;
		if (size1 == size2) {
			val1 = l1.val;
			val2 = l2.val;
			next = recur(l1.next, size1 - 1, l2.next, size2 - 1);
		} else if (size1 < size2) {
			val2 = l2.val;
			next = recur(l1, size1, l2.next, size2 - 1);
		} else {
			val1 = l1.val;
			next = recur(l1.next, size1 - 1, l2, size2);
		}

		int remain = (val1 + val2 + carry) % 10;
		carry = (val1 + val2 + carry) / 10;

		ListNode current = new ListNode(remain);
		current.next = next;

		return current;
	}
```

# 相似题目
- [Problem2-Add Two Numbers](2022-10-21-leetcode-problem-2.md)