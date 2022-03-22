# 题目链接
https://leetcode-cn.com/problems/permutation-in-string/

# 解答过程
这个题目和Problem438几乎是一样的，相比Problem438还更简单些，只需要判断是否存在子串即可，不再赘述。

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

		if (Arrays.equals(target, slide)) {
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
				if (Arrays.equals(target, slide)) {
					return true;
				}
			}
		}

		return false;
	}
```