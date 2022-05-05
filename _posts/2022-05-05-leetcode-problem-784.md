# 题目链接
https://leetcode-cn.com/problems/letter-case-permutation/

# 解答过程
这个题目用回溯的范式来套的话，还是比较容易的，不多赘述了。

```java
	public List<String> letterCasePermutation(String s) {
		List<String> result = new ArrayList<>();
		char[] charArr = s.toCharArray();
		backtrack(result, 0, charArr);

		return result;
	}

	private void backtrack(List<String> result, int curPos, char[] charArr) {
		if (curPos == charArr.length) {
			result.add(new String(charArr));
			return;
		}

		char curChar = charArr[curPos];

		if (Character.isLetter(curChar)) {
			charArr[curPos] = Character.toLowerCase(curChar);
			backtrack(result, curPos + 1, charArr);

			charArr[curPos] = Character.toUpperCase(curChar);
			backtrack(result, curPos + 1, charArr);
		} else {
			backtrack(result, curPos + 1, charArr);
		}
	}
```