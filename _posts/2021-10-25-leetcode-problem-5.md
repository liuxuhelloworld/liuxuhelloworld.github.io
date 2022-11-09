# 题目链接
https://leetcode-cn.com/problems/longest-palindromic-substring/

# 解答过程
## 动态规划
比较典型的动态规划问题，应该不难想出状态转移方程吧，挺早以前写的题了，记不清当时写得是不是顺利。

```java
    public String longestPalindromeSubstring(String s) {
        if (s == null) {
            return null;
        }

        int len = s.length();

        String ret = "";

        // dp[i][j] represents whether s[i..j] is a palindromic substring
        boolean[][] dp = new boolean[len][len];

        for (int step = 0; step < len; step++) {
            for (int i = 0; i < len-step; i++) {
                if (step == 0) {
                    // s[i] is a palindromic substring of length one
                    dp[i][i] = true;
                } else if (step == 1) {
                    // s[i..i+1] is a palindromic substring of length two if s[i] == s[i+1]
                    dp[i][i+1] = s.charAt(i) == s.charAt(i+1);
                } else {
                    // s[i..i+step] is a palindromic substring of length step
                    // if s[i] == s[i+step] and s[i+1..i+step-1] is also a palindromic substring
                    dp[i][i+step] = dp[i+1][i+step-1] && s.charAt(i) == s.charAt(i+step);
                }

                if (dp[i][i+step] && step+1 > ret.length()) {
                    ret = s.substring(i, i+step+1);
                }
            }
        }

        return ret;
    }
```

## 暴力解法
我觉得这个所谓中心扩展解法就是暴力解法。。。，先以每个元素自身为中心左右扩展，再以相邻元素相等的组合为中心左右扩展。

```java
    public String longestPalindromeSubstring(String s) {
        if (s == null) {
            return null;
        }

        int len = s.length();

        String ret = "";

        for (int i = 0; i < len; i++) {
            String expanded = expand(s, i, i);
            if (expanded.length() > ret.length()) {
                ret = expanded;
            }

            if (i > 0 && s.charAt(i) == s.charAt(i-1)) {
                expanded = expand(s, i-1, i);
                if (expanded.length() > ret.length()) {
                    ret = expanded;
                }
            }
        }

        return ret;
    }

    private String expand(String s, int left, int right) {
        int len = s.length();
        int i = left - 1, j = right + 1;
        while (i >= 0 && j < len) {
            if (s.charAt(i) != s.charAt(j)) {
                break;
            }
            i--;
            j++;
        }

        return s.substring(i+1, j);
    }
```

# 相似题目
- [Problem9-Palindrome Number](2022-11-08-leetcode-problem-9.md)
- [Problem125-Valid Palindrome](2022-11-09-leetcode-problem-125.md)