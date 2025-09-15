# 题目链接
https://leetcode.cn/problems/two-sum/

# 解答过程
对于这个题目，双循环的暴力解法是很显然的一个思路，时间复杂度O(N^2)。要想降低时间复杂度，就需要以更快的速度判断出某个值是否存在，不仅是否存在，如果存在还需要知道这个值在原数组中的下标，由此，需要使用哈希表。

```java
	public int[] twoSum(int[] nums, int target) {
		Map<Integer, Integer> map = new HashMap<>(nums.length);

		for (int i = 0; i < nums.length; i++) {
			int minus = target - nums[i];
			if (map.containsKey(minus)) {
				return new int[] {map.get(minus), i};
			}

			map.put(nums[i], i);
		}

		return new int[0];
	}
```