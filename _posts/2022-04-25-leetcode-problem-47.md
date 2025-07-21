# 题目链接
https://leetcode-cn.com/problems/permutations-ii/

# 解答过程
这个题目是Problem46的进阶版，在Problem46中，限定入参数组是不包含重复元素的，这样就不需要关心重复解问题。Problem47允许入参数组包含重复元素，那么单纯地像Problem46那样做回溯处理就不行了，需要在回溯的过程中避免重复解。如何避免重复解呢？假设某个重复元素X有n个，那么对于每一个全排列，就有n个位置是这个X值。重复解怎么来的？对于这n个位置的X，不管你怎么交换，结果还是一样的全排列，这就是重复解的来源。重点来了，要想避免重复解，需要确保这n个X之间不做任何交换，换句话说，需要确保这n个X只以一种方式被填入进去，一种可行的方法是确保这n个X是从左到右按顺序被填入进去，从而避免重复解。

另外，这个题目避免重复解的处理和Problem15、Problem16、Problem18有异曲同工之处，可以比较并体会。

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