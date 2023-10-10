# 题目链接
https://leetcode-cn.com/problems/3sum/
当
# 解答过程
题目本身很好理解，给定一个数组，找出所有相加之和为0的三元组，注意还需要对三元组去重。

如果从暴力解法入手，直接三重循环，是很容易找出所有满足条件的三元组的，但一方面这种做法时间效率较差，另一方面如何对三元组去重呢？

把这个题目和Problem1、Problem167放到一起来看。先排序，排序以后循环处理，每次循环固定一个元素，从剩余元素中按Problem167中的双指针解法确定另两个元素。这样一个解体思路，如果综合Problem1、Problem167一起来看，倒也并不显得很突兀。但是，坦白讲，这种解法我也是参考官方题解写的。一开始，我的直觉是这个题目是不是考察深度优先遍历或者回溯......。如果有Problem1、Problem167打底，一步步来，只能说，倒也并不显得很突兀。

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

# 相似题目
- [Problem1-Two Sum](2022-10-12-leetcode-problem-1.md)
- [Problem167-Two Sum](2021-08-06-leetcode-problem-167.md)
- [Problem18-Four Sum](2021-11-24-leetcode-problem-18.md)
- [Problem16-Three Sum Closest](2021-11-23-leetcode-problem-16.md)