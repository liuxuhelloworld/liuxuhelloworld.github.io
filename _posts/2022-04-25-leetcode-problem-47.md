# 题目链接
https://leetcode-cn.com/problems/permutations-ii/

# 解答过程
这个题目是Problem46的进阶版，在Problem46中，限定入参数组是不包含重复元素的，这样就不需要关心重复解问题。Problem47允许入参数组包含重复元素，那么单纯地像Problem46那样做回溯处理就不行了，需要在回溯的过程中避免重复解。如何避免重复解，我应该是挺久以前参考官方题解写的。简单来说，对于重复值元素，确保他们是从左到右按顺序被选中进入permutation就可以避免重复解。另外，对于Problem46中不使用标记数组的写法，我尝试了下，没写出来，感觉不适用于那种写法不适用于Problem47，有时间再研究下。

```java
	public List<List<Integer>> permuteUnique(int[] nums) {
		List<List<Integer>> result = new ArrayList<>();

		List<Integer> permutation = new ArrayList<>();

		boolean[] tag = new boolean[nums.length];

		Arrays.sort(nums);

		backtrack(result, permutation, nums, tag, 0);

		return result;
	}

	private void backtrack(List<List<Integer>> result, List<Integer> permutation, int[] nums, boolean[] tag, int cur) {
		if (cur == nums.length) {
			result.add(new ArrayList<>(permutation));
			return;
		}

		for (int i = 0; i < nums.length; i++) {
			if (tag[i] == true
				|| (i > 0 && nums[i] == nums[i-1] && !tag[i-1])) {
				continue;
			}

			tag[i] = true;
			permutation.add(nums[i]);
			backtrack(result, permutation, nums, tag, cur+1);
			permutation.remove(permutation.size()-1);
			tag[i] = false;
		}
	}
```