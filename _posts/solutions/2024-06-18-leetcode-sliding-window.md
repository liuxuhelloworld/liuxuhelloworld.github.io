<a id="problem_3"></a>
# 问题3 无重复字符的最长子串 ★☆☆☆☆
https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/

滑动窗口的解法可以从暴力解法的思路中得到启发。比如说使用暴力解法，我们遍历n次，每次以s[i]为起点开始遍历直到碰到了重复字符s[j]，本次遍历结束，然后开始以s[i+1]为起点开始下一轮遍历。这样操作的问题在哪呢？就在于下一轮遍历以s[i+1]为起点，这样就太笨了。因为s[i]到s[j-1]是确定没有重复字符的，假设s[j]和s[i]到s[j-1]中的某个s[k]是重复的，那么至少应该从s[k+1]开始下一轮遍历，这样就减少了很多重复计算。

```java
    public int lengthOfLongestSubstringWithoutRepeatingCharacters(String s) {
        int len = s.length();

        Map<Character, Integer> index = new HashMap<>();

        int max = 0;
        int left = 0;
        for (int right = 0; right < len; right++) {
            char c = s.charAt(right);
            if (index.containsKey(c)) {
                int lastIndex = index.get(c);
                if (lastIndex + 1 > left) {
                    left = lastIndex + 1;
                }
            }
            index.put(c, right);
            max = Math.max(max, right - left + 1);
        }

        return max;
    }
```

# 问题438 找到字符串中所有字母异位词 ★★☆☆☆
https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/

这个题目可以从暴力解法的思路入手，即顺序遍历所有子串，判断每个子串是不是p的anagram. 第一个关键点，如何判断一个子串是不是p的anagram呢？比较直接的思路是使用两个计数数组，分别计数两个字符串中每个字母的个数，然后比较这两个数组的计数是否相等。第二个关键点，如何优化计算效率呢？题目标签滑动窗口具有一定的启发性，顺序遍历子串时后一个子串的计数可以基于前一个子串的计数而来，就像一个滑动的窗口，每次右移一位，吃进右侧字母的同时吐出左侧字母。

```java
	public List<Integer> findAnagrams(String s, String p) {
		List<Integer> ret = new ArrayList<>();

		int plen = p.length();
		int slen = s.length();
		if (slen < plen) {
			return ret;
		}

		int[] target = new int[26];
		int[] rolling = new int[26];

		for (int i = 0; i < plen; i++) {
			target[p.charAt(i) - 'a']++;
			rolling[s.charAt(i) - 'a']++;
		}
		if (compare(target, rolling)) {
			ret.add(0);
		}

		for (int i = 1; i <= s.length() - plen; i++) {
			if (s.charAt(i-1) != s.charAt(i+plen-1)) {
				rolling[s.charAt(i-1) - 'a']--;
				rolling[s.charAt(i+plen-1) - 'a']++;
			}

			if (compare(target, rolling)) {
				ret.add(i);
			}
		}

		return ret;
	}

	private boolean compare(int[] target, int[] rolling) {
		for (int i = 0; i < 26; i++) {
			if (target[i] != rolling[i]) {
				return false;
			}
		}

		return true;
	}
```

# 问题567 字符串的排列 ★★☆☆☆
https://leetcode-cn.com/problems/permutation-in-string/

这个题目和问题438几乎是一样的，相比问题438还更简单些，只需要判断是否存在子串即可，不再赘述。

```java
	public boolean checkInclusion(String s1, String s2) {
		int len1 = s1.length();
		int len2 = s2.length();

		if (len1 > len2) {
			return false;
		}

		int[] target = new int[26];
		int[] slide = new int[26];
		for (int i = 0; i < len1; i++) {
			target[s1.charAt(i) - 'a']++;
			slide[s2.charAt(i) - 'a']++;
		}

		if (Arrays.equals(target, slide)) {
			return true;
		} else {
			for (int i = 1; i <= len2 - len1; i++) {
				char out = s2.charAt(i - 1);
				char in = s2.charAt(i + len1 - 1);
				if (out == in) {
					continue;
				}
				slide[out - 'a']--;
				slide[in - 'a']++;
				if (Arrays.equals(target, slide)) {
					return true;
				}
			}
		}

		return false;
	}
```

# 问题209 长度最小的子数组 ★★☆☆☆
https://leetcode-cn.com/problems/minimum-size-subarray-sum/

这个题目可以算是比较典型的滑动窗口题目了，滑动窗口的解法理解起来并不难，循环不变量是，每次循环先右指针向右扩大窗口，持续扩大直到当前窗口内元素之和大于等于target，然后开始左指针向右缩小窗口，持续缩小直到当前窗口内元素之和小于target。在这个过程中，及时更新满足条件的最小子数组长度。

```java
	public int minSubArrayLen(int target, int[] nums) {
		int min = Integer.MAX_VALUE;
		int left = 0, right = 0;
		int sum = 0;

		while (right < nums.length) {
			sum += nums[right];

			while (sum >= target) {
				min = Math.min(min, right - left + 1);
				sum -= nums[left];
				left++;
			}

			right++;
		}

		return min == Integer.MAX_VALUE ? 0 : min;
	}
```

# 问题713 乘积小于K的子数组 ★★☆☆☆
https://leetcode-cn.com/problems/subarray-product-less-than-k/

这个题目其实和问题209很像，利用滑动窗口来解答的思路都是，先向右扩展右指针，直到左右指针窗口内的元素不再满足某个要求，开始向右扩展左指针，在这个过程中，维护一个目标计算结果。

```java
	public int numSubarrayProductLessThanK(int[] nums, int k) {
		if (k <= 1) {
			return 0;
		}

		int cnt = 0;

		int left = 0, right = 0, product = 1;
		while (right < nums.length) {
			product *= nums[right];

			while (product >= k) {
				product /= nums[left];
				left++;
			}

			cnt += right - left + 1;

			right++;
		}

		return cnt;
	}
```

# 问题187 重复的DNA序列 ★★★☆☆
https://leetcode-cn.com/problems/repeated-dna-sequences/

在知晓题目主标签是sliding window的条件下，还是比较容易想到通过滑动窗口来降低计算成本。但是具体怎样将一个固定长度的子串映射为某个特征值呢？一开始我想的是把这个子串当作一个26进制的数来计算，但是很快发现溢出问题。后面参考了官方题解才想到，为了保证计算速度，往2进制靠拢才是正确思路。

直接计算子串的哈希值，然后判断是否相同子串，这样可以吗？理论上，这样是不可以的，因为可能出现哈希碰撞，即相同子串的哈希值相等，但哈希值相等的子串不一定是相同子串。

另外，为什么题目背景是DNA序列呢？把字符串限定在由ACGT组成，其实就是为了简化二进制编码，只需要2位二进制位就可以完成对ACGT的编码。其实，即便题目放开为全部英文字母，这个题目的解决方法也是一样的，无非是需要5位二进制位才能编码26个英文字母。

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