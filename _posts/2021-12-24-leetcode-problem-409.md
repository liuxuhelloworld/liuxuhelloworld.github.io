# 题目链接
https://leetcode-cn.com/problems/longest-palindrome/

# 解答过程
这个题目要求计算由一串大小写字符能构成的最长回文串的长度，那么比较显然的想法，一个字符满足什么条件就可以算作回文串的一部分从而贡献自己的一份力量呢？如果这个字符有另一个和它一样的字符就可以嘛。换句话说，如果一个字符的总数cnt是偶数，那么它们全部可以作为回文串的组成部分。那么如果一个字符的总数cnt是奇数呢？那么cnt-1一定是偶数，一样可以作为回文串的一部分。这样，我们可能剩下的就是一堆个数为1的字符了，这些字符没有办法再配对了，但是它们当中可以有一个代表作为回文串的中间元素，这样我们就计算出了能组成的最长回文串的长度。

```java
	public int longestPalindrome(String s) {
		if (s == null || s.isEmpty()) {
			return 0;
		}

		int[] cnt = new int[52];

		for (int i = 0; i < s.length(); i++) {
			char ch = s.charAt(i);
			if (ch >= 'a' && ch <= 'z') {
				cnt[ch - 'a']++;
			}
			if (ch >= 'A' && ch <= 'Z') {
				cnt[ch - 'A' + 26]++;
			}
		}

		int max = 0;
		boolean hasSingle = false;
		for (int i = 0; i < cnt.length; i++) {
			if (cnt[i] % 2 == 0) {
				max += cnt[i];
				cnt[i] = 0;
			} else {
				max += cnt[i] - 1;
				cnt[i] = 1;
				hasSingle = true;
			}
		}

		if (hasSingle) {
			max += 1;
		}

		return max;
	}
```

瞅了一眼官方题解，思路是差不多的，我的代码比官方题解略繁琐，但我觉得更好理解。