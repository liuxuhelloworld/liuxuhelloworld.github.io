# 题目链接
https://leetcode-cn.com/problems/3sum-closest/

# 解答过程
在Problem15三数之和的基础上，这个题目就比较好写了。

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