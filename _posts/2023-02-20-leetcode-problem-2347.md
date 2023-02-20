# 题目链接
https://leetcode.cn/problems/best-poker-hand/

# 解答过程
这个题目确实比较简单了，根据题目要求，Flush只需要判断花色，Three of a Kind、Pair、High Card这些只需要判断出现频次最高的点数。

```java
	public String bestHand(int[] ranks, char[] suits) {
		boolean isFlush = isFlush(suits);
		if (isFlush) {
			return "Flush";
		}

		int[] count = new int[14]; // 0 index is not used
		for (int i = 0; i < ranks.length; i++) {
			count[ranks[i]]++;
		}

		int max = 0;
		for (int i = 0; i < count.length; i++) {
			if (count[i] > max) {
				max = count[i];
			}
		}

		if (max >= 3) {
			return "Three of a Kind";
		} else if (max == 2) {
			return "Pair";
		} else {
			return "High Card";
		}
	}

	private boolean isFlush(char[] suits) {
		for (int i = 1; i < suits.length; i++) {
			if (suits[i] != suits[i-1]) {
				return false;
			}
		}

		return true;
	}
```