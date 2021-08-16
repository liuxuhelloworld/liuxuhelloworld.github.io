# 题目链接
https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/

# 解答过程
## 动态规划
这个题目首先想到的是动态规划，定义dp[i]表示以s[i]结尾的具有最大非重复字符的子串的长度，那么dp[i]的最大值就是整个字符串中具有最大非重复字符的子串的长度。那么在dp[i]的基础上如何计算dp[i+1]呢？关键就是要确认s[i+1]这个字符在dp[i]对应的那个具有最大非重复字符的子串中的位置，如果s[i+1]不在其中，那么dp[i+1]=dp[i]+1，否则需要求差计算。

```java
    public int lengthOfLongestSubstringWithoutRepeatingCharacters(String s) {
        int len = s.length();
        if (len == 0) {
            return 0;
        }

        // dp[i] represents the length of longest substring that end with s[i] and has no repeating characters
        int[] dp = new int[len];
        dp[0] = 1;
        int max = 1;
        for (int i = 1; i < len; i++) {
            int last = lastIndexOf(s, i-dp[i-1],i-1, s.charAt(i));
            if (last == -1) {
                dp[i] = dp[i-1] + 1;
            } else {
                dp[i] = i-last;
            }
            if (dp[i] > max) {
                max = dp[i];
            }
        }

        return max;
    }

    private int lastIndexOf(String s, int start, int end, char c) {
        for (int i = end; i >= start; i--) {
            if (s.charAt(i) == c) {
                return i;
            }
        }

        return -1;
    }
```