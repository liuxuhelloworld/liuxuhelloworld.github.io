# 题目链接
https://leetcode.cn/problems/utf-8-validation/

# 解答过程
这个题目坦白讲我觉得够不上中等难度，主要是解题思路比较直接。基于给定的UTF-8编码规则，判断一个byte array是不是一串有效的UTF-8字符，那么很直接的，就挨个字符遍历判断嘛。当遍历到某个位置时，判断当前字节是不是满足四种编码方式之一，如果不满足，那么显然不是有效的UTF-8编码字符串了。如果满足某一种，那么还需要细分，如果是满足单字节编码的要求，那么显然当前字节自己就构成一个有效的UTF-8编码，那么继续判断下一个位置即可；如果是满足双字节编码的要求，那么需要进一步判断下面一个位置是不是一个合法的trailing byte；同理，如果是满足三字节或者四字节编码的要求，那么需要进一步判断下面两个或者下面三个位置是不是合法的trailing byte。

```java
	private static final int UTF_8_1_BYTE_MASK = 0b10000000;
	private static final int UTF_8_2_BYTES_MASK = 0b11100000;
	private static final int UTF_8_2_BYTES_TARGET = 0b11000000;
	private static final int UTF_8_3_BYTES_MASK = 0b11110000;
	private static final int UTF_8_3_BYTES_TARGET = 0b11100000;
	private static final int UTF_8_4_BYTES_MASK = 0b11111000;
	private static final int UTF_8_4_BYTES_TARGET = 0b11110000;
	private static final int UTF_8_TRAILING_BYTE_MASK = 0b11000000;
	private static final int UTF_8_TRAILING_BYTE_TARGET = 0b10000000;

	public boolean validUtf8(int[] data) {
		int startIdx = 0;
		do {
			startIdx = nextStartIdx(data, startIdx);
			if (startIdx == -1) {
				break;
			}
		} while (startIdx < data.length);

		if (startIdx == -1) {
			return false;
		}

		return true;
	}

	private int nextStartIdx(int[] data, int startIdx) {
		int cur = data[startIdx];

		if ((cur & UTF_8_1_BYTE_MASK) == 0) {
			return startIdx + 1;
		}

		if ((cur & UTF_8_2_BYTES_MASK) == UTF_8_2_BYTES_TARGET) {
			if (isConsecutiveTrailingByte(data, startIdx + 1, 1)) {
				return startIdx + 2;
			}
		}

		if ((cur & UTF_8_3_BYTES_MASK) == UTF_8_3_BYTES_TARGET) {
			if (isConsecutiveTrailingByte(data, startIdx + 1, 2)) {
				return startIdx + 3;
			}
		}

		if ((cur & UTF_8_4_BYTES_MASK) == UTF_8_4_BYTES_TARGET) {
			if (isConsecutiveTrailingByte(data, startIdx + 1, 3)) {
				return startIdx + 4;
			}
		}

		return -1;
	}

	private boolean isConsecutiveTrailingByte(int[] data, int startIdx, int num) {
		for (int i = 0; i < num; i++) {
			int curIdx = startIdx + i;
			if (curIdx >= data.length) {
				return false;
			}

			int cur = data[curIdx];
			if ((cur & UTF_8_TRAILING_BYTE_MASK) != UTF_8_TRAILING_BYTE_TARGET) {
				return false;
			}
		}

		return true;
	}
```

如果一定要说这个题目的难点的话，我觉得主要考察的是对字符编码和位运算的掌握情况，以及基本的编码能力。