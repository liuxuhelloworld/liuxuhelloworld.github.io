# 题目链接
https://leetcode-cn.com/problems/string-to-integer-atoi/

# 解答过程
这个题目是经典的字符串转整数，从解题思路上来说，并不复杂，根据题目要求按顺序处理字符串就可以，需要注意的是如何处理overflow的问题。相比Problem7，这个题目并没有要求不能使用长整型，所以偷懒的做法就是直接用长整型。如果题目要求不能使用长整型，那么就需要使用和Problem7中类似的方法，在overflow之前去判断，特别的，需要考虑到32位整数取值范围的非对称性。

```java
    public int myAtoi(String s) {
        char[] chars = s.toCharArray();

        int i = 0;
        while (i < chars.length && Character.isSpaceChar(chars[i])) {
            i++;
        }

        if (i == chars.length
            || (chars[i] != '+' && chars[i] != '-' && !Character.isDigit(chars[i]))) {
            return 0;
        }

        boolean negative = false;
        if (chars[i] == '+' || chars[i] == '-') {
            negative = chars[i] == '-';
            i++;
        }

        long ret = 0;
        while (i < chars.length && Character.isDigit(chars[i])) {
            ret = ret * 10 + (chars[i]-'0');
            if (ret > Integer.MAX_VALUE) {
                break;
            }
            i++;
        }
        ret = ret * (negative ? -1 : 1);

        if (ret > Integer.MAX_VALUE) {
            return Integer.MAX_VALUE;
        } else if (ret < Integer.MIN_VALUE) {
            return Integer.MIN_VALUE;
        } else {
            return (int) ret;
        }
    }
```

相似题目
- [Problem7-Reverse Integer](2022-11-15-leetcode-problem-7.md)