<a id="problem_5"></a>
# 问题5 最长回文子串 ★☆☆☆☆
https://leetcode-cn.com/problems/longest-palindromic-substring/

动态规划的解法见[问题5](2024-04-08-leetcode-dynamic-programming.md#problem_5)

我觉得所谓中心扩展解法就是暴力解法。。。，先以每个元素自身为中心左右扩展，再以相邻元素相等的组合为中心左右扩展。

```java
    public String longestPalindromeSubstring(String s) {
        if (s == null) {
            return null;
        }

        int len = s.length();

        String ret = "";

        for (int i = 0; i < len; i++) {
            String expanded = expand(s, i, i);
            if (expanded.length() > ret.length()) {
                ret = expanded;
            }

            if (i > 0 && s.charAt(i) == s.charAt(i-1)) {
                expanded = expand(s, i-1, i);
                if (expanded.length() > ret.length()) {
                    ret = expanded;
                }
            }
        }

        return ret;
    }

    private String expand(String s, int left, int right) {
        int len = s.length();
        int i = left - 1, j = right + 1;
        while (i >= 0 && j < len) {
            if (s.charAt(i) != s.charAt(j)) {
                break;
            }
            i--;
            j++;
        }

        return s.substring(i+1, j);
    }
```

<a id="problem_6"></a>
# 问题6 Z字形变换 ★☆☆☆☆
https://leetcode-cn.com/problems/zigzag-conversion/

我觉得这种问题最直接的处理就是构造一个二维数组，遍历字符串，挨个将字符放到对应的位置上，然后再收集起来即可。看了下官方题解，思路其实是类似的，不过官方题解的代码看起来更简洁：使用List\<StringBuilder\>替换二维数组，从而不用再计算列数；在边界处理上下转向。至于通过公式直接寻址构造，我觉得没必要。

```java
    public String convert(String s, int numRows) {
        int len = s.length();
        if (len == 0) {
            return "";
        }

        numRows = Math.min(numRows, len);
        if (numRows == 1) {
            return s;
        }

        List<StringBuilder> rows = new ArrayList<>(numRows);
        for (int i = 0; i < numRows; i++) {
            rows.add(new StringBuilder());
        }

        boolean goingDown = false;

        int curRow = 0;
        for (int k = 0; k < len; k++) {
            rows.get(curRow).append(s.charAt(k));

            if (curRow == 0 || curRow == numRows-1) {
                goingDown = !goingDown;
            }

            curRow += goingDown ? 1 : -1;
        }

        StringBuilder ret = new StringBuilder();
        for (StringBuilder row : rows) {
            ret.append(row);
        }

        return ret.toString();
    }
```

<a id="problem_9"></a>
# 问题9 回文数 ☆☆☆☆☆
https://leetcode.cn/problems/palindrome-number/

显然，如果入参是负数，那么就不是回文数。对于非负数，通过转换得到一个char数组的话，很容易判断是不是回文。但是题目里明确提示，有没有办法不做字符串转换？对于一个整数，从低位到高位遍历每一位数字是很容易的，那么如何在这样一个遍历过程中判断是否回文呢？我们在从低位到高位遍历的过程中，维护一个新的逆序的整数值，根据回文的特征，如果原整数是回文的，那么最终得到的新的逆序的整数和原整数应该相等。

```java
	public boolean isPalindrome(int x) {
		if (x < 0) {
			return false;
		}

		int y = x, reversed = 0;
		while (y != 0) {
			int remain = y % 10;
			reversed = reversed*10 + remain;
			y = y / 10;
		}

		return x == reversed;
	}
```

<a id="problem_43"></a>
# 问题43 字符串相乘 ★☆☆☆☆
https://leetcode.cn/problems/multiply-strings/

这个题目乍一看有点难度，我能想到的就是模拟竖式乘法，把字符串整数相乘拆解为字符串整数和数字相乘，然后进行字符串相加。在这个过程当中，要注意右侧补0等问题。

这个题目可以看作[问题415](#problem_415)的升级版，问题415是字符串整数相加，这里改为相乘。

简单看了下官方题解，官方题解给出的另外两种解法当然都更高级一些，尤其是卷积法。这些解法，如果没有相关经验或相关提示，并且用纸笔写写画画做推导，我觉得是不容易写出来的，有时间再研究下。

```java
	public String multiply(String num1, String num2) {
		int len1 = num1.length();
		int len2 = num2.length();

		if (len1 == 0 || len2 == 0) {
			throw new IllegalArgumentException();
		}

		String multiplicand, multiplier;
		if (len1 < len2) {
			multiplicand = num2;
			multiplier = num1;
		} else {
			multiplicand = num1;
			multiplier = num2;
		}

		if (multiplier.equals("0") || multiplicand.equals("0")) {
			return "0";
		}

		String leftShift = "";
		String result = "";
		for (int i = multiplier.length()-1; i >= 0; i--) {
			int ch = multiplier.charAt(i) - '0';

			result = addTwoStrings(result, multiplyByDigit(multiplicand, ch) + leftShift);
			leftShift += "0";
		}

		return result;
	}

	private String multiplyByDigit(String num, int digit) {
		if (!(digit >= 1 && digit <= 9)) {
			return "";
		}

		StringBuilder builder = new StringBuilder();
		int carry = 0;
		for (int i = num.length() - 1; i >= 0; i--) {
			int ch = num.charAt(i) - '0';

			int product = ch * digit + carry;
			int remainder = product % 10;
			carry = product / 10;

			builder.append((char)(remainder + '0'));
		}
		if (carry != 0) {
			builder.append((char)(carry + '0'));
		}

		return builder.reverse().toString();
	}

	private String addTwoStrings(String num1, String num2) {
		StringBuilder builder = new StringBuilder();

		String reversedNum1 = new StringBuilder(num1).reverse().toString();
		String reversedNum2 = new StringBuilder(num2).reverse().toString();

		int len1 = reversedNum1.length();
		int len2 = reversedNum2.length();

		int i, j;
		int carry = 0;
		for (i = 0, j = 0; i < len1 || j < len2; i++, j++) {
			int ch1, ch2;

			if (i < len1) {
				ch1 = reversedNum1.charAt(i) - '0';
			} else {
				ch1 = 0;
			}
			if (j < len2) {
				ch2 = reversedNum2.charAt(j) - '0';
			} else {
				ch2 = 0;
			}

			int sum = ch1 + ch2 + carry;

			int remainder = sum % 10;
			carry = sum / 10;
			builder.append((char)(remainder + '0'));
		}

		if (carry != 0) {
			builder.append((char)(carry + '0'));
		}

		return builder.reverse().toString();
	}
```

<a id="problem_125"></a>
# 问题125 验证回文串 ☆☆☆☆☆
https://leetcode.cn/problems/valid-palindrome/

这个题目确实是比较简单了，基本的字符串处理操作，或者先做过滤和转换，或者直接由两端向中间遍历，逻辑都差不多，不赘述。

```java
	public boolean isPalindrome(String s) {
		StringBuilder builder = new StringBuilder();

		for (int i = 0; i < s.length(); i++) {
			char ch = s.charAt(i);
			if ((ch >= 'a' && ch <= 'z')
				|| (ch >= '0' && ch <= '9')) {
				builder.append(ch);
			} else if (ch >= 'A' && ch <= 'Z') {
				builder.append((char)(ch - 'A' + 'a'));
			}
		}

		char[] chars = builder.toString().toCharArray();
		for (int i = 0, j = chars.length - 1; i < j; i++, j--) {
			if (chars[i] != chars[j]) {
				return false;
			}
		}

		return true;
	}
```

<a id="problem_409"></a>
# 问题409 最长回文串 ☆☆☆☆☆
https://leetcode-cn.com/problems/longest-palindrome/

这个题目要求计算由一串大小写字符能构成的最长回文串的长度，那么比较显然的想法，一个字符满足什么条件就可以算作回文串的一部分从而贡献自己的一份力量呢？如果这个字符有另一个和它一样的字符就可以嘛。换句话说，如果一个字符的总数cnt是偶数，那么它们全部可以作为回文串的组成部分。那么如果一个字符的总数cnt是奇数呢？那么cnt-1一定是偶数，一样可以作为回文串的一部分。这样，我们可能剩下的就是一堆个数为1的字符了，这些字符没有办法再配对了，但是它们当中可以有一个代表作为回文串的中间元素，这样我们就计算出了能组成的最长回文串的长度。

```java
	public int longestPalindrome(String s) {
		if (s == null || s.isEmpty()) {
			return 0;
		}

		int[] cnt = new int[52];

		for (int i = 0; i < s.length(); i++) {
			char ch = s.charAt(i);
			if (ch >= 'a' && ch <= 'z') {
				cnt[ch - 'a']++;
			}
			if (ch >= 'A' && ch <= 'Z') {
				cnt[ch - 'A' + 26]++;
			}
		}

		int max = 0;
		boolean hasSingle = false;
		for (int i = 0; i < cnt.length; i++) {
			if (cnt[i] % 2 == 0) {
				max += cnt[i];
				cnt[i] = 0;
			} else {
				max += cnt[i] - 1;
				cnt[i] = 1;
				hasSingle = true;
			}
		}

		if (hasSingle) {
			max += 1;
		}

		return max;
	}
```

<a id="problem_415"></a>
# 题目415 字符串相加 ☆☆☆☆☆
https://leetcode.cn/problems/add-strings/

这个题目主要考察基本的字符串操作能力，比较简单了，从题目要求以及参数限制来看，显然转换为整数再相加是行不通的，况且，不使用库函数的话，字符串转整数比这个题目本身还要难一些。这个题目就是逆序遍历字符串，挨个位置相加，注意处理进位即可。

这个题目和[问题2](2024-06-12-leetcode-linked-list.md#problem_2)、[问题445](2024-06-12-leetcode-linked-list.md#problem_445)相比，最大的不同之处就在于字符串是可以随机存取的，无论顺序遍历还是逆序遍历，都好处理。

```java
	public String addStrings(String num1, String num2) {
		StringBuilder builder = new StringBuilder();

		int len1 = num1.length();
		int len2 = num2.length();

		int carry = 0;
		for (int i = len1 - 1, j = len2 - 1; i >= 0 || j >= 0; i--, j--) {
			int val1 = 0;
			if (i >= 0) {
				val1 = num1.charAt(i) - '0';
			}
			int val2 = 0;
			if (j >= 0) {
				val2 = num2.charAt(j) - '0';
			}

			int sum = val1 + val2 + carry;
			int remain = sum % 10;
			carry = sum / 10;

			builder.append((char) (remain + '0'));
		}

		if (carry == 1) {
			builder.append("1");
		}

		return builder.reverse().toString();
	}
```

