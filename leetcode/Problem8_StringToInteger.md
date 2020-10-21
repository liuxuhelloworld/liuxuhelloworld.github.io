# 题目链接

https://leetcode-cn.com/problems/string-to-integer-atoi/

# 解法1
思路很简答， 就是遍历字符串。注意，累加数值时要及时退出循环，防止溢出。
执行用时超过50%的java提交记录。

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
            return (int)ret;
        }
}
```

