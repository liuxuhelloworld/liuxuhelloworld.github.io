# 题目链接
https://leetcode-cn.com/problems/permutation-in-string/

# 解答过程
我的思路是先用一个map统计目标字符串s1的各个字符数量。然后遍历s2，如果当前遍历到的字符在s1中，则map的计数减1，遍历s1.len个字符后判断map计数是否全为0。后面看了官方题解，才明白这个题目标签是sliding window的原因。

```java
	public boolean checkInclusion(String s1, String s2) {
		int len1 = s1.length();
		int len2 = s2.length();

		if (len1 > len2) {
			return false;
		}

		int[] target = new int[26];
		int[] slide = new int[26];
		for (int i = 0; i < len1; i++) {
			target[s1.charAt(i) - 'a']++;
			slide[s2.charAt(i) - 'a']++;
		}

		if (isSame(target, slide)) {
			return true;
		} else {
			for (int i = 1; i <= len2 - len1; i++) {
				char out = s2.charAt(i - 1);
				char in = s2.charAt(i + len1 - 1);
				if (out == in) {
					continue;
				}
				slide[out - 'a']--;
				slide[in - 'a']++;
				if (isSame(target, slide)) {
					return true;
				}
			}
		}

		return false;
	}

	private boolean isSame(int[] arr1, int[] arr2) {
		for (int i = 0; i < arr1.length; i++) {
			if (arr1[i] != arr2[i]) {
				return false;
			}
		}

		return true;
	}
```