<a id="problem_2"></a>
# 问题2 两数相加 ★☆☆☆☆
https://leetcode.cn/problems/add-two-numbers/description/

这个题目本身难度不大，主要考察的是基本的链表操作能力，与合并两个有序链表有一定相似之处。

```java
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(-1), tail = dummy;
        int carry = 0;
        while (l1 != null || l2 != null) {
            int val1 = l1 != null ? l1.val : 0;
            int val2 = l2 != null ? l2.val : 0;

            int val = (val1 + val2 + carry) % 10;
            carry = (val1 + val2 + carry) / 10;
            tail.next = new ListNode(val);
            tail = tail.next;

            if (l1 != null) {
                l1 = l1.next;
            }
            if (l2 != null) {
                l2 = l2.next;
            }
        }

        if (carry == 1) {
            tail.next = new ListNode(1);
        }

        return dummy.next;
    }
```

<a id="problem_445"></a>
# 问题445 两数相加 ★☆☆☆☆
https://leetcode.cn/problems/add-two-numbers-ii/

这个题目和[问题2](#problem_2)相似，区别在于问题2里链表是倒序表示的整数，这里链表是正序表示的整数。由于整数加法是从最低位到最高位处理，所以问题2相对简单，只需要注意进位问题就可以了。那么比较直接的想法，就是把链表做反转，相加以后再反转得到最终结果。然而题目中也指出，有没有解法可以不使用反转。

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