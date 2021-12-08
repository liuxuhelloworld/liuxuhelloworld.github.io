# 题目链接
https://leetcode-cn.com/problems/repeated-dna-sequences/

# 解答过程
在知晓题目主标签是sliding window的条件下，还是比较容易想到通过滑动窗口来降低计算成本。但是具体怎样将一个固定长度的子串映射为某个特征值呢？一开始我想的是把这个子串当作一个26进制的数来计算，但是很快发现溢出问题。后面参考了官方题解才想到，为了保证计算速度，往2进制靠拢才是正确思路。

```java
	private Map<Character, Integer> map = new HashMap<Character, Integer>() {{
		put('A', 0b00);
		put('C', 0b01);
		put('G', 0b10);
		put('T', 0b11);
	}};

	public List<String> findRepeatedDnaSequences(String s) {
		if (s == null) {
			return Collections.emptyList();
		}

		int len = s.length();
		if (len <= 10) {
			return Collections.emptyList();
		}

		List<String> ret = new ArrayList<>();

		Map<Integer, Integer> cnt = new HashMap<>();

		int hash = 0;
		for (int i = 0; i < 10; i++) {
			hash = hash << 2 | map.get(s.charAt(i));
		}
		cnt.put(hash, 1);

		for (int i = 1; i <= len - 10; i++) {
			char in = s.charAt(i + 9);
			hash = hash << 2 | map.get(in);
			hash = hash & 0b00000000000011111111111111111111;

			cnt.put(hash, cnt.getOrDefault(hash, 0) + 1);

			if (cnt.get(hash) == 2) {
				ret.add(s.substring(i, i + 10));
			}
		}

		return ret;
	}
```