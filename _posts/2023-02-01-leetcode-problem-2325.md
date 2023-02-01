# 题目链接
https://leetcode.cn/problems/decode-the-message/

# 解答过程
这个题目比较简单了，关键是先维护出一个解密table，不赘述。

```java
	public String decodeMessage(String key, String message) {
		char[] table = new char[26];
		for (int i = 0; i < 26; i++) {
			table[i] = 0;
		}

		char[] keyCharArr = key.toCharArray();
		char ch = 'a';
		for (int i = 0; i < keyCharArr.length; i++) {
			char cur = keyCharArr[i];
			if (cur != ' ' && table[cur - 'a'] == 0) {
				table[cur - 'a'] = ch;
				ch = (char)(ch + 1);
			}
		}

		char[] messageCharArr = message.toCharArray();
		char[] decodedCharArr = new char[messageCharArr.length];
		for (int i = 0; i < messageCharArr.length; i++) {
			char cur = messageCharArr[i];
			if (cur != ' ') {
				decodedCharArr[i] = table[cur - 'a'];
			} else {
				decodedCharArr[i] = ' ';
			}
		}

		return String.valueOf(decodedCharArr);
	}
```