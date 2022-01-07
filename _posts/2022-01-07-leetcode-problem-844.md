# 题目链接
https://leetcode-cn.com/problems/backspace-string-compare/

# 解答过程
这个题目最直接的思路就是按照backspace规则，分别把两个字符串做backspace处理，然后比较得到的结果。更进一步的说，挨个遍历字符串的字符，如果是小写字母，则读进来；如果是\#，则把最近一个读进来的字符删掉。

```java
	public boolean backspaceCompare(String s, String t) {
		String backspaced1 = backspace(s);
		String backspaced2 = backspace(t);

		if (backspaced1.equals(backspaced2)) {
			return true;
		} else {
			return false;
		}
	}

	private String backspace(String s) {
		char[] chars = s.toCharArray();

		StringBuilder builder = new StringBuilder();

		for (int i = 0; i < chars.length; i++) {
			if (Character.isLowerCase(chars[i])) {
				builder.append(chars[i]);
			} else {
				if (builder.length() > 0) {
					builder.deleteCharAt(builder.length() - 1);
				}
			}
		}

		return builder.toString();
	}
```

做题前我知道这个题目的标签是双指针，但是实在没想到怎么用双指针，看了下官方题解才明白，有点意思，抽时间再写下双指针解法。