# 题目链接
https://leetcode-cn.com/problems/hand-of-straights/

# 解答过程
解答这个题目的思路如下：
首先，根据题目要求，如果hand包含的元素值个数不能被groupSize整除，那么显然不能满足分组要求。
那么接下来该如何判断是否能满足分组要求呢？我的直觉是先计数，统计每个元素出现的次数。然后我只能想到比较暴力的解法就是按元素值从小到大，以groupSize为单元，扣减计数。如果处理到某个元素时当前groupSize大小的单元内不足扣减，或者全部处理完后还有元素的计数大于0，那么就不能满足分组要求，否则就可以满足分组要求。

```java
	public boolean isNStraightHand(int[] hand, int groupSize) {
		int len = hand.length;
		if (len % groupSize != 0) {
			return false;
		}

		Map<Integer, Integer> cnt = new TreeMap<>();
		for (int i = 0; i < len; i++) {
			cnt.put(hand[i], cnt.getOrDefault(hand[i], 0) + 1);
		}

		for (Map.Entry<Integer, Integer> entry : cnt.entrySet()) {
			int key = entry.getKey();
			int val = entry.getValue();

			if (val == 0) {
				continue;
			}

			cnt.put(key, 0);
			for (int j = 1; j < groupSize; j++) {
				if (!cnt.containsKey(key + j)
					|| cnt.get(key + j) < val) {
					return false;
				} else {
					cnt.put(key + j, cnt.get(key + j) - val);
				}
			}
		}

		return cnt.values().stream()
			.allMatch(e -> e == 0);
	}
```

提交以后是可以通过的，但是效率比较差，另外，这种解法看起来就怪怪的。

看了下官方题解，思路上有一些共通之处，但是官方题解明显更优雅一些，更能体现greedy的思想，后面抽时间再写下这个思路的代码吧。