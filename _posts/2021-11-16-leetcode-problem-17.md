# 题目链接
https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/

# 解答过程
这个题目从最简单的思路入手，我们进行N重循环就可以嘛，每次循环从当前数字对应的可选字符集合里挑一个出来，这样就得到了所有可能的组合。那么怎么写一个N重循环呢，N是runtime时确认的值。典型的写法是通过递归来实现N重循环。这种通过递归来遍历所有可能的方法称为回溯法（backtrack）。

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