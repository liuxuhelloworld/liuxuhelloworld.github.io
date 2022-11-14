# 题目链接
https://leetcode-cn.com/problems/zigzag-conversion/

# 解答过程
这个题目也是挺久以前写的了。我觉得这种问题最直接的处理就是构造一个二维数组，遍历字符串，挨个将字符放到对应的位置上，然后再收集起来即可。看了下官方题解，思路其实是类似的，不过官方题解的代码看起来更简洁：使用List\<StringBuilder\>替换二维数组，从而不用再计算列数；在边界处理上下转向。至于通过公式直接寻址构造，我觉得没必要，除非真的是关系到效率的生产代码。从toy code的角度来说，公式的版本看起来无趣得多。

```java
    public String convert(String s, int numRows) {
        int len = s.length();
        if (len == 0) {
            return "";
        }

        numRows = Math.min(numRows, len);
        if (numRows == 1) {
            return s;
        }

        List<StringBuilder> rows = new ArrayList<>(numRows);
        for (int i = 0; i < numRows; i++) {
            rows.add(new StringBuilder());
        }

        boolean goingDown = false;

        int curRow = 0;
        for (int k = 0; k < len; k++) {
            rows.get(curRow).append(s.charAt(k));

            if (curRow == 0 || curRow == numRows-1) {
                goingDown = !goingDown;
            }

            curRow += goingDown ? 1 : -1;
        }

        StringBuilder ret = new StringBuilder();
        for (StringBuilder row : rows) {
            ret.append(row);
        }

        return ret.toString();
    }
```