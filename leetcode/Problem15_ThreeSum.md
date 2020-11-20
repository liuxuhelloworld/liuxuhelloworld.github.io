# 题目链接

https://leetcode-cn.com/problems/3sum/

# 解法1
参考官方题解的思路写的，个人感觉比官方题解的代码清晰易懂一点。基本思路就是，先排序，然后在锁定a元素的情况下，利用有序性，双指针遍历b和c。需要注意的是，锁定a和锁定b的时候要去重。为什么没有对c做去重？因为curSum == target时，k--的同时j++，所以对b做去重隐含着对c也做了去重。

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