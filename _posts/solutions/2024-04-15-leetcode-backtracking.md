# 问题93 复原IP地址
https://leetcode-cn.com/problems/restore-ip-addresses/

这个题目一开始我用的是一种简单粗暴的笨办法，就是循环遍历，得到所有可能的分割结果，然后筛选出符合IP地址格式的结果。这种方法先不说时间效率，代码看起来就繁琐不美观。

其实这个题目算是比较典型的回溯法题目，用回溯法的解体模板去套的话，并不难写出来。回溯遍历，每次遍历时，分别尝试1位、2位、3位作为新的part添加到维护的ipParts中，注意只有合法的part才有必要处理，非法的直接忽略就可以了。当遍历到最后一位时，判断当前ipParts是不是恰好有4个元素，如果是，说明得到了一个有效的IP地址。另外，通过判断ipParts的元素个数和当前剩余的字符数，我们可以提前剪枝。

```java
	public List<String> restoreIpAddresses(String s) {
		List<String> ipAddresses = new ArrayList<>();
		List<String> ipParts = new ArrayList<>();

		backtrack(ipAddresses, ipParts, s, 0);

		return ipAddresses;
	}

	private void backtrack(List<String> ipAddresses,
	                       List<String> ipParts,
	                       String s,
	                       int idx) {

		if (idx == s.length()) {
			if (ipParts.size() == 4) {
				ipAddresses.add(String.join(".", ipParts));
			}
			return;
		}

		if (ipParts.size() >= 4) {
			return;
		}

		if (ipParts.size() == 3 && s.length() - idx > 3) {
			return;
		}

		for (int i = 0; i < 3 && idx+i+1 <= s.length(); i++) {
			String part = s.substring(idx, idx + i + 1);
			if (isValidIpPart(part)) {
				ipParts.add(part);
				backtrack(ipAddresses, ipParts, s, idx + i + 1);
				ipParts.remove(ipParts.size() - 1);
			}
		}
	}

	private boolean isValidIpPart(String part) {
		int len = part.length();

		if (len == 1) {
			return true;
		}

		char first = part.charAt(0);
		if (first == '0') {
			return false;
		}

		if (len == 3) {
			return part.compareTo("255") <= 0;
		}

		return true;
	}
```