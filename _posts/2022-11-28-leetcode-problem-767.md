# 题目链接
https://leetcode.cn/problems/reorganize-string/

# 解答过程
刚入手这个题目时，有点想当然了，直觉上的思路是先计数字符个数，然后按个数由大到小，往重组后的字符串里塞，当找不到可塞的位置时，认为是bad cacse. 提交以后不通过，很快就发现了问题在哪。一个简单的例子，假设个数最少的字符是两个，其他字符都很好的塞到了重组后字符串的最左端，现在还剩下两个位置，那么就会处理成bad case, 但其实这并不是bad case. 所以，该怎么解决这个问题呢？其实这个时候，我们只需要把上一轮的字符往后塞就可以了。OK，当前轮处理发现进行不下去了，需要回到上一轮重新调整，这不就是回溯么？所以，如果一开始就从回溯的角度考虑，思路就很容易打开了，甚至不需要计数字符个数，就是遍历原字符串，挨个往重组后的字符串里塞，塞不下去了就往前回溯，处理到原字符串末尾时说明已经得到了一个符合条件的重组字符串。按照这个思路写了一版之后提交，发现还是没通过，超时了......。看了一下超时case，应该是bad case导致的递归深度太大。在最外层加了一个可行性判断，回溯时也稍微优化了下，改为每次取剩余计数最多的字符优先处理。提交通过，虽然效率还是一般。

```java
	public String reorganizeString(String s) {
		char[] origin = s.toCharArray();

		int maxCount = 0, allCount = origin.length;

		int[] count = new int[26];
		for (int i = 0; i < origin.length; i++) {
			int idx = origin[i] - 'a';
			count[idx]++;

			if (count[idx] > maxCount) {
				maxCount = count[idx];
			}
		}

		if (allCount - maxCount < maxCount - 1) {
			return "";
		}

		char[] reorganized = new char[origin.length];

		return backtrack(reorganized, count);
	}

	private String backtrack(char[] reorganized, int[] count) {
		int idx = getNonZeroMaxCount(count);
		if (idx == -1) {
			return String.valueOf(reorganized);
		}

		char ch = (char) ('a' + idx);

		for (int i = 0; i < reorganized.length; i++) {
			if (reorganized[i] != 0) {
				continue;
			}

			if (i > 0
				&& reorganized[i-1] == ch) {
				continue;
			}

			if (i < reorganized.length-1
				&& reorganized[i+1] == ch) {
				continue;
			}

			reorganized[i] = ch;
			count[idx]--;

			String s = backtrack(reorganized, count);
			if (!s.isEmpty()) {
				return s;
			}

			count[idx]++;
			reorganized[i] = 0;
		}

		return "";
	}

	private int getNonZeroMaxCount(int[] count) {
		int idx = -1;
		int max = 0;
		for (int i = 0; i < count.length; i++) {
			if (count[i] > max) {
				max = count[i];
				idx = i;
			}
		}

		return idx;
	}
```