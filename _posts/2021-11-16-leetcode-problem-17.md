# 题目链接
https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/

# 解答过程
## 解法1
这个题目是挺久以前写的了，当时应该是看了官方题解后写的。从代码看，使用backtracking还是很好理解的。

```java
	private static final Map<Character, String> PHONE_NUMBER_TO_CHARACTERS = new HashMap<Character, String>()
	{{
		put('2', "abc");
		put('3', "def");
		put('4', "ghi");
		put('5', "jkl");
		put('6', "mno");
		put('7', "pqrs");
		put('8', "tuv");
		put('9', "wxyz");
	}};

	public List<String> letterCombinations(String digits) {
		int len = digits.length();
		if (len == 0) {
			return Collections.emptyList();
		}

		List<String> ret = new ArrayList<>();
		backtrack(ret, digits, 0, new StringBuffer());

		return ret;
	}

	private void backtrack(List<String> combinations, String digits, int index, StringBuffer combination) {
		if (index == digits.length()) {
			combinations.add(combination.toString());
		} else {
			char digit = digits.charAt(index);
			String chars = PHONE_NUMBER_TO_CHARACTERS.get(digit);
			for (int i = 0; i < chars.length(); i++) {
				combination.append(chars.charAt(i));
				backtrack(combinations, digits, index+1, combination);
				combination.deleteCharAt(index);
			}
		}
	}
```

## 解法2
递归的写法应该是当时自己写出来的吧？实在记不清了，反正刚刚整理这个题目时脑子里一片空白，递归写法也没想起来。。。

```java
	private static final Map<Character, String> PHONE_NUMBER_TO_CHARACTERS = new HashMap<Character, String>()
	{{
		put('2', "abc");
		put('3', "def");
		put('4', "ghi");
		put('5', "jkl");
		put('6', "mno");
		put('7', "pqrs");
		put('8', "tuv");
		put('9', "wxyz");
	}};

	public List<String> letterCombinations(String digits) {
		return letterCombinationRecur(digits, 0, digits.length()-1);
	}

	private List<String> letterCombinationRecur(String digits, int start, int end) {
		if (start > end) {
			return Collections.emptyList();
		}

		char digit = digits.charAt(start);
		String chars = PHONE_NUMBER_TO_CHARACTERS.get(digit);
		List<String> tails = letterCombinationRecur(digits, start + 1, end);
		List<String> combinations = new ArrayList<>();
		for (char ch : chars.toCharArray()) {
			if (tails.size() > 0) {
				for (String tail : tails) {
					combinations.add(ch + tail);
				}
			} else {
				combinations.add(ch + "");
			}
		}

		return combinations;
	}
```