# 题目链接
https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/

# 解答过程
这个题目可以从暴力解法的思路入手，即顺序遍历所有子串，判断每个子串是不是p的anagram. 第一个关键点，如何判断一个子串是不是p的anagram呢？比较直接的思路是使用两个计数数组，分别计数两个字符串中每个字母的个数，然后比较这两个数组的计数是否相等。第二个关键点，如何优化计算效率呢？题目标签滑动窗口具有一定的启发性，顺序遍历子串时后一个子串的计数可以基于前一个子串的计数而来，就像一个滑动的窗口，每次右移一位，吃进右侧字母的同时吐出左侧字母。

```java
	public List<Integer> findAnagrams(String s, String p) {
		List<Integer> ret = new ArrayList<>();

		int plen = p.length();
		int slen = s.length();
		if (slen < plen) {
			return ret;
		}

		int[] target = new int[26];
		int[] rolling = new int[26];

		for (int i = 0; i < plen; i++) {
			target[p.charAt(i) - 'a']++;
			rolling[s.charAt(i) - 'a']++;
		}
		if (compare(target, rolling)) {
			ret.add(0);
		}

		for (int i = 1; i <= s.length() - plen; i++) {
			if (s.charAt(i-1) != s.charAt(i+plen-1)) {
				rolling[s.charAt(i-1) - 'a']--;
				rolling[s.charAt(i+plen-1) - 'a']++;
			}

			if (compare(target, rolling)) {
				ret.add(i);
			}
		}

		return ret;
	}

	private boolean compare(int[] target, int[] rolling) {
		for (int i = 0; i < 26; i++) {
			if (target[i] != rolling[i]) {
				return false;
			}
		}

		return true;
	}
```