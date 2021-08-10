# 题目链接
https://leetcode-cn.com/problems/reverse-words-in-a-string-iii/

# 解答过程
额，这个题感觉没啥可说的，我用了比较多库函数，效率一般，但我觉得这样比较简洁。

```java
	public String reverseWords(String s) {
		String[] words = s.split(" ");
		String[] reversed = Arrays.stream(words)
			.map(e -> reverse(e))
			.collect(Collectors.toList())
			.toArray(new String[0]);

		return String.join(" ", reversed);
	}

	private String reverse(String s) {
		StringBuilder rev = new StringBuilder();
		for (int i = s.length()-1; i >= 0; i--) {
			rev.append(s.charAt(i));
		}
		return rev.toString();
	}
```