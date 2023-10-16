# 题目链接
https://leetcode.cn/problems/add-two-numbers-ii/

# 解答过程
这个题目和Problem2相似，区别在于Problem2里链表是倒序表示的整数，Problem445里链表是正序表示的整数。由于整数加法是从最低位到最高位处理，所以Problem2相对简单，只需要注意进位问题就可以了。那么比较直接的想法，就是把Problem445的链表做反转，相加以后再反转得到最终结果。然而题目中也指出，有没有解法可以不使用反转。

这个地方我一开始想到的是递归解法，代码提交以后也能通过，从代码来看，其实还比较好理解，只是进位的问题处理起来有点难看，使用了一个成员变量来存储递归过程中每次计算后的进位值。

后面看了官方题解，额，有一种恍然大悟的感觉。使用栈确实是更合乎逻辑、代码更简洁的解法。

```java
	public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
		Stack<Integer> stack1 = new Stack<>();
		while (l1 != null) {
			stack1.push(l1.val);
			l1 = l1.next;
		}

		Stack<Integer> stack2 = new Stack<>();
		while (l2 != null) {
			stack2.push(l2.val);
			l2 = l2.next;
		}

		int carry = 0;
		ListNode dummy = new ListNode(-1);

		while (!stack1.isEmpty()
			|| !stack2.isEmpty()) {

			int val1 = 0, val2 = 0;
			if (!stack1.isEmpty()) {
				val1 = stack1.pop();
			}
			if (!stack2.isEmpty()) {
				val2 = stack2.pop();
			}

			int remain = (val1 + val2 + carry) % 10;
			carry = (val1 + val2 + carry) / 10;

			ListNode node = new ListNode(remain);
			node.next = dummy.next;
			dummy.next = node;
		}

		if (carry != 0) {
			ListNode node = new ListNode(1);
			node.next = dummy.next;
			dummy.next = node;
		}

		return dummy.next;
	}
```

# 相似题目
- [Problem2-Add Two Numbers](2022-10-21-leetcode-problem-2.md)
- [Problem415-Add Strings](2023-07-17-leetcode-problem-415.md)
- [Problem43-Multiply Strings](2023-10-16-leetcode-problem-43.md)