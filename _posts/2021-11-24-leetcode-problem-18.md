# 题目链接
https://leetcode-cn.com/problems/4sum/

# 解答过程
这个题目和Problem15非常像了，那么类似的，解题的思路也是一致的，只不过比Problem15多一层循环。就像在Problem15的题解里写的，如果基于Problem1 --> Problem167 --> Problem15 --> Problem18这样一个顺序来层层递进，那么解题的思路是有脉络可循的。但是如果上来就是直接写这样一个题目，我感觉会比较懵，不容易有清晰的思路。

```java
	public List<List<Integer>> fourSum(int[] nums, int target) {
		int len = nums.length;
		if (len < 4) {
			return Collections.emptyList();
		}

		List<List<Integer>> ret = new ArrayList<>();

		Arrays.sort(nums);

		for (int i = 0; i < len; i++) {
			if (i > 0 && nums[i] == nums[i-1]) {
				continue;
			}
			for (int j = i+1; j < len; j++) {
				if (j > i+1 && nums[j] == nums[j-1]) {
					continue;
				}
				int sum = nums[i] + nums[j];
				if (sum > target && nums[j] > 0) {
					break;
				} else {
					int diff = target - sum;
					int m = j+1, n = len-1;
					while (m < n) {
						if (m > j+1 && nums[m] == nums[m-1]) {
							m++;
							continue;
						}
						int curSum = nums[m] + nums[n];
						if (curSum < diff) {
							m++;
						} else if (curSum > diff) {
							n--;
						} else {
							ret.add(Arrays.asList(nums[i], nums[j], nums[m], nums[n]));
							m++;
							n--;
						}
					}
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
- [Problem16-Three Sum Closest](2021-11-23-leetcode-problem-16.md)