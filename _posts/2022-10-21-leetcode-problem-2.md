# 题目链接
https://leetcode.cn/problems/add-two-numbers/description/

# 解答过程
这个题目本身难度不大，主要考察的是基本的链表操作能力，与合并两个有序链表有一定相似之处。有两个值得注意的点，一个是dummy head，一个是**while (l1 != null || l2 != null)**这种遍历方式。

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

# 相似题目
- [Problem445-Add Two Numbers](2022-10-27-leetcode-problem-445.md)