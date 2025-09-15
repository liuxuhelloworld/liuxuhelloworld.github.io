# 题目链接
https://leetcode-cn.com/problems/subsets/

# 解答过程
这个题目和Problem77、Problem39、Problem46这些题目都有一定关联性。Problem77是计算n个元素的所有k元组，即所有包含k个元素的子集，Problem78是计算所有子集。Problem39是计算元素之和等于tagert的子集，Problem78是计算所有子集。Problem46是计算n个元素的所有全排列，Problem78是计算所有子集。以上这些题目回溯的思路和代码都颇有相似之处，可以比较并体会。

```java
	public List<List<Integer>> subsets(int[] nums) {
		List<List<Integer>> subsets = new ArrayList<>();
		List<Integer> subset = new ArrayList<>();

		backtrack(subsets, subset, nums, 0);

		return subsets;
	}

	private void backtrack(List<List<Integer>> subsets, List<Integer> subset, int[] nums, int cur) {
		if (cur == nums.length) {
			subsets.add(new ArrayList<>(subset));
			return;
		}

		// cur excluded
		backtrack(subsets, subset, nums, cur + 1);

		// cur included
		subset.add(nums[cur]);
		backtrack(subsets, subset, nums, cur + 1);
		subset.remove(subset.size() - 1);
	}
```