# 题目链接

https://leetcode-cn.com/problems/4sum/

# 解法1
和三数之和的解法类似，先排序，然后固定两个数，剩下两个数双指针遍历。如果前两个数的和已经大于目标值，并且已经进入正数序列，则跳出循环。

PS: 看了下官方题解，还可以增加一些剪枝策略。

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