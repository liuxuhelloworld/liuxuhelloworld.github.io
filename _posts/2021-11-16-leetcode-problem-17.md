# 题目链接
https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/

# 解答过程
## Recursion
从递归的角度考虑，还是比较容易提取出子问题的，不赘述。

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

## Backtracking
从backtracking的角度考虑，从代码来看也不难理解。我觉得这种backtracking方法的declaration具备一定的普遍性，即一个结果集、一个当前维护中的结果、回溯的对象、回溯进度标识。

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