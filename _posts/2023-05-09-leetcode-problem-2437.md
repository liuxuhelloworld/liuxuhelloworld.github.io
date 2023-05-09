# 题目链接
https://leetcode.cn/problems/number-of-valid-clock-times/

# 解答过程
这个题目是比较简单了，如果考编码时出类似题目，应该是绿灯放行的意思了吧。虽然题目的解答思路不难，但我觉得这个题目还是能考察下代码能力的，尤其是能不能把代码写得简单清晰易理解。

```java
	public int countTime(String time) {
		char[] chars = time.toCharArray();
		boolean hourHighQuestionMark = chars[0] == '?';
		boolean hourLowQuestionMark = chars[1] == '?';
		boolean minuteHighQuestionMark = chars[3] == '?';
		boolean minuteLowQuestionMark = chars[4] == '?';

		int hours = 1;
		if (hourHighQuestionMark && hourLowQuestionMark) {
			hours = 24;
		} else {
			if (hourHighQuestionMark) {
				if (chars[1] > '3') {
					hours = 2;
				} else {
					hours = 3;
				}
			} else if (hourLowQuestionMark) {
				if (chars[0] < '2') {
					hours = 10;
				} else {
					hours = 4;
				}
			}
		}

		int minutes = 1;
		if (minuteHighQuestionMark) {
			minutes *= 6;
		}
		if (minuteLowQuestionMark) {
			minutes *= 10;
		}

		return hours * minutes;
	}
```