# 问题136 只出现一次的数字 ★☆☆☆☆
https://leetcode-cn.com/problems/single-number/

这个题目本质上是考察二进制位操作，利用v ^ v = 0的特性，很容易想到：

```java
	public int singleNumber(int[] nums) {
		int ret = 0;

		for (int num : nums) {
			ret ^= num;
		}

		return ret;
	}
```	

# 问题190 颠倒二进制位 ★☆☆☆☆
https://leetcode-cn.com/problems/reverse-bits/

这个题目的解题思路很直接，通过右移循环取出最低位的二进制位，然后通过左移累积结果值即可。

那么题目额外提出的，如果涉及大量调用，如何优化效率呢？一开始我没什么思路，看了一下官方题解，这个所谓的“位运算分治”，看描述还好理解，但是代码看不懂......

我又思考了下，既然要提高效率，那就每次多处理几位嘛。比如，每次取最低位的2位二进制位，这样循环次数就减少了一半，但是需要注意，累计结果值时，需要把取到的2位二进制翻转。那么有什么办法能快速翻转2位二进制位呢？一开始我的入手点是有没有什么二进制运算能完成，但是直觉上这是条死路。转念一想，直接弄个mapping就可以了嘛，把00,01,10,11对应的翻转存到数组里。进一步的，既然可以每次取最低位的2位，那么干脆更进一步，每次取最低位的4位呢？这样循环次数进一步减少，4位二进制位对应的mapping数组也不难处理。那么，还能进一步每次取最低位的8位吗？额，那就得维护一个包含256个元素的mapping数组了，感觉有点没必要了。

```java
	private int[] mapping = {0b0000, 0b1000, 0b0100, 0b1100, 0b0010, 0b1010, 0b0110, 0b1110,
							 0b0001, 0b1001, 0b0101, 0b1101, 0b0011, 0b1011, 0b0111, 0b1111};

	public int reverseBits(int n) {
		int ret = 0;

		for (int i = 0; i < 8; i++) {
			int rightest4Bits = n & 15;
			n = n >> 4;
			ret = ret << 4;
			ret = ret | mapping[rightest4Bits];
		}

		return ret;
	}
```

# 问题191 位1的个数 ★☆☆☆☆
https://leetcode-cn.com/problems/number-of-1-bits/

这个题目最简单的解法就是循环取最低位的二进制位，判断是1还是0，累加计数。需要注意的一点是，Java不支持无符号整数，右移二进制位时要做逻辑右移。

对于题目中额外提出的，如何能优化效率呢？对于整型编码具备一定认知的话，可以通过n&(n-1)来操作，每次n&(n-1)会抹掉低位的一个二进制位1，直到n变为0.

```java
	public int hammingWeight(int n) {
		if (n == 0) {
			return 0;
		}

		int weight = 0;
		do {
			weight++;
		} while ((n = (n & n-1)) != 0);


		return weight;
	}
```

# 问题231 2的幂 ★☆☆☆☆
https://leetcode-cn.com/problems/power-of-two/

2的幂有什么特点呢？2的幂的二进制表示中只有一个二进制位1，因此，利用n&(n-1)很容易判断是否是2的幂。

```java
	public boolean isPowerOfTwo(int n) {
		return n > 0 && (n & (n-1)) == 0;
	}
```

# 问题401 二进制手表
https://leetcode-cn.com/problems/binary-watch/

这个题目我一开始的思路是回溯法，把题目要求抽象为从一个包含N个元素的数组中，挑选M个数出来，要求它们的和满足一定限制，返回所有可能的挑选对应的和的集合。然后吭哧吭哧写了一堆代码，正确性没有问题，但是效率较差。

看了一眼官方题解才恍然大悟，其实题目描述已经很明显的是二进制位运算问题，搞不懂为什么会先想到回溯......。从二进制位运算的思路入手，就是给定两个数值，一个小时和一个分钟，判断这俩数值包含的二进制1一共有多少位，如果符合目标值，那么就是时间结果集中的一个。

```java
	public List<String> readBinaryWatch(int turnedOn) {
		List<String> list = new ArrayList<>();

		int[] binaryOneCnt = new int[64];
		for (int i = 0; i < 64; i++) {
			binaryOneCnt[i] = cntBinaryOne(i);
		}

		StringBuilder builder = new StringBuilder();

		for (int hour = 0; hour <= 11; hour++) {
			int hourBinaryOneCnt = binaryOneCnt[hour];
			for (int minute = 0; minute <= 59; minute++) {
				int minuteBinaryOneCnt = binaryOneCnt[minute];
				if (hourBinaryOneCnt + minuteBinaryOneCnt == turnedOn) {
					builder.setLength(0);
					builder.append(hour);
					builder.append(":");
					builder.append(minute < 10 ? "0" + minute : minute);
					list.add(builder.toString());
				}
			}
		}

		return list;
	}

	private int cntBinaryOne(int val) {
		int cnt = 0;
		while (val != 0) {
			cnt++;
			val &= (val - 1);
		}

		return cnt;
	}
```

# 问题393 UTF-8编码验证 ★☆☆☆☆
https://leetcode.cn/problems/utf-8-validation/

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

# 问题89 格雷编码 ★★☆☆☆
https://leetcode-cn.com/problems/gray-code/

这个题目本质上还是位操作，关键点是如果我们有了n-1位的格雷编码，如何得到n位的格雷编码呢？顺序遍历挨个高位补一位二进制位0，结果和n-1位的格雷编码一样。接着，倒序遍历挨个高位补一位二进制位1，并追加到n-1位格雷编码的末尾。可以推断，这样得到的结果是符合要求的格雷编码。

```java
public List<Integer> grayCodeV2(int n) {
		assert n >= 1 && n <= 16;

		List<Integer> res = new ArrayList<>();
		res.add(0);

		for (int i = 0; i < n; i++) {
			int op = 1 << i;
			int len = res.size();
			for (int j = len-1; j >= 0; j--) {
				res.add(res.get(j) | op);
			}
		}

		return res;
}
```