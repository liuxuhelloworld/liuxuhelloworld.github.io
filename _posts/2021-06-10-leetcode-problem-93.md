# 题目链接
https://leetcode-cn.com/problems/restore-ip-addresses/

# 解答过程
这个题目我能想到的就是笨办法：维护一个结果集S(n)，S(n)表示长度为n的子串对应的合法或部分合法的ip值划分，那么如何计算S(n+1)呢？尝试将第n+1个字符作为独立的ip part追加到S(n)，尝试将第n+1个字符作为S(n)的最后一个part的一部分追加到S(n)，这样就得到了S(n+1)。最后，过滤掉最终结果集中的非法ip值。

```java
	public List<String> restoreIpAddresses(String s) {
		assert s != null;

		int len = s.length();
		if (len == 0) {
			return Collections.emptyList();
		}

		List<String> candidates = new ArrayList<>();

		String first = s.substring(0, 1);
		candidates.add(first);

		for (int i = 1; i < len; i++) {
			String cur = s.substring(i, i+1);

			List<String> newCandidates = new ArrayList<>();

			for (String candidate : candidates) {
				if (canBeAppended(candidate)) {
					newCandidates.add(candidate + "." + cur);
				}

				if (canBeAppendedAsWhole(candidate, cur)) {
					newCandidates.add(candidate + cur);
				}
			}

			candidates = newCandidates;
		}

		return candidates.stream()
			.filter(this::isValidIPAddress)
			.collect(Collectors.toList());
	}

	private boolean canBeAppended(String candidate) {
		int dotNum = countDotNum(candidate);
		if (dotNum <= 2) {
			return true;
		} else {
			return false;
		}
	}

	private boolean canBeAppendedAsWhole(String candidate, String appended) {
		int index = candidate.lastIndexOf('.');

		String lastPart;
		if (index == -1) {
			lastPart = candidate;
		} else {
			lastPart = candidate.substring(candidate.lastIndexOf('.') + 1);
		}

		if (lastPart.equals("0")) {
			return false;
		}

		int val = Integer.parseInt(lastPart + appended);
		if (val > 0 && val <= 255) {
			return true;
		} else {
			return false;
		}
	}

	private boolean isValidIPAddress(String candidate) {
		int dotNum = countDotNum(candidate);

		if (dotNum == 3) {
			return true;
		} else {
			return false;
		}
	}

	private int countDotNum(String candidate) {
		int dotNum = 0;
		for (int i = 0; i < candidate.length(); i++) {
			if (candidate.charAt(i) == '.') {
				dotNum++;
			}
		}

		return dotNum;
	}
```

我觉得这个思路还是比较好理解的，结果也是正确的，不过时间和空间效率都不高。

看了下官方题解，官方题解推荐的是回溯法，没太仔细看，反正给我的感觉是不那么容易理解，后面抽时间再研究下。