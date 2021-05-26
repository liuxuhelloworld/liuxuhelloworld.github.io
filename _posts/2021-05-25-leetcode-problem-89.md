# 题目链接
https://leetcode-cn.com/problems/gray-code/

# 解法1
一开始面对这个题目，没有什么思路，只想到比较暴力的深度优先。维持一个list存储当前累加的结果，一个当前处理对象。下一个对象怎么得到呢？既然相邻元素只能有1位二进制位不同，那就从低位到高位遍历，每次改1个二进制位，新值如果已经在list中，就尝试下一个位置，如果不在list中，就作为下一个对象，进行下一次递归。当list中元素个数已经满了，直接返回。

```java
public List<Integer> grayCodeV1(int n) {
		assert n >= 1 && n <= 16;

		List<Integer> grayCodes = new ArrayList<>();

		int limit = 1;
		for (int i = 1; i <= n; i++) {
			limit *= 2;
		}

		Integer cur = 0;
		grayCodes.add(cur);

		dfs(grayCodes, cur, n, limit);

		return grayCodes;
	}

	private void dfs(List<Integer> bits, Integer cur, int n, int limit) {
		if (bits.size() == limit) {
			return;
		}

		for (int i = 0; i < n; i++) {
			Integer oneBitChange = cur ^ (1 << i);

			if (!bits.contains(oneBitChange)
				&& bits.size() < limit) {
				bits.add(oneBitChange);
				dfs(bits, oneBitChange, n, limit);
			}
		}
	}
```

这种解法从正确性上应该是没问题的，感觉也比较容易理解，但是问题是效率较差，有太多重复计算，提交后果然超时。

# 解法2
超时报错后研究了下代码，感觉也没什么地方可以再优化了。耐心耗尽，直接看了题解，才意识到应该使用动态规划。那么从S(n)怎么得到S(n+1)呢？S(n)顺序挨个高位补1位二进制0，结果和S(n)一样，接着，倒序挨个高位补1位二进制1，并追加到S(n)末尾，这样就得到了S(n+1)

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
倒序遍历高位补1这个思路，我觉得还是有点难度的，不太容易一下想到。