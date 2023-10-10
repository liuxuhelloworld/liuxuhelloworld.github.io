# 题目链接
https://leetcode-cn.com/problems/3sum-closest/

# 解答过程
这个题目和Problem15几乎一样，只是这里是计算最接近目标值的三元组的和是多少。从直觉上，我觉得这种最值问题给人的感觉更难。对于这个题目，解题思路和Problem15是一致的，不赘述。还是想提一下的是，基于Problem1 --> Problem167 --> Problem15 --> Problem16这样一个顺序层层梳理下来，解题思路就显得”水到渠成“，否则，死记硬背也容易忘记。

```java
	public int threeSumClosest(int[] nums, int target) {
		int len = nums.length;
		if (len == 0) {
			throw new RuntimeException("bad input");
		}

		Arrays.sort(nums);

		int ret = 0;
		int minDiff = Integer.MAX_VALUE;
		for (int i = 0; i < len; i++) {
			if (i > 0 && nums[i] == nums[i - 1]) {
				continue;
			}

			int j = i+1, k = len-1;
			while (j < k) {
				if (j > i + 1 && nums[j] == nums[j - 1]) {
					j++;
					continue;
				}

				int curSum = nums[i] + nums[j] + nums[k];

				int curDiff = Math.abs(curSum - target);
				if (curDiff < minDiff) {
					ret = curSum;
					minDiff = curDiff;
				}

				if (curSum > target) {
					k--;
				} else if (curSum < target) {
					j++;
				} else {
					return curSum;
				}
			}
		}

		return ret;
	}
```

# 相似题目
- [Problem1-Two Sum](2022-10-12-leetcode-problem-1.md)
- [Problem167-Two Sum](2021-08-06-leetcode-problem-167.md)
- [Problem15-Three Sum](2021-11-22-leetcode-problem-15.md)
- [Problem18-Four Sum](2021-11-24-leetcode-problem-18.md)