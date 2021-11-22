# 题目链接
https://leetcode-cn.com/problems/3sum/

# 解答过程
这个题目当时应该也是参考官方题解写的。对这个题目，我的直觉是做回溯或者深度优先遍历。。。算了，背诵全文吧。

```java
	public List<List<Integer>> threeSum(int[] nums) {
		int len = nums.length;
		if (len == 0) {
			return Collections.emptyList();
		}

		Arrays.sort(nums);

		List<List<Integer>> ret = new ArrayList<>();
		for (int i = 0; i < len; i++) {
			if (i > 0 && nums[i] == nums[i-1]) {
				continue;
			}

			int target = 0 - nums[i];
			int j = i + 1, k = len - 1;
			while (j < k) {
				if (j > i+1 && nums[j] == nums[j-1]) {
					j++;
					continue;
				}

				int curSum = nums[j] + nums[k];
				if (curSum > target) {
					k--;
				} else if (curSum < target) {
					j++;
				} else {
					ret.add(Arrays.asList(nums[i], nums[j], nums[k]));
					j++;
					k--;
				}
			}
		}

		return ret;
	}
```