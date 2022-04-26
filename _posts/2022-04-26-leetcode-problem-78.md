# 题目链接
https://leetcode-cn.com/problems/subsets/

# 解答过程
这个题目和Problem39很相似，Problem39是计算元素之和等于tagert的子集，Problem78是计算所有子集，回溯的思路是一致的，回溯到某个元素时，要么选中这个元素并继续回溯，要么不选这个元素并继续回溯。

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

看了下官方题解，通过二进制位取子集的迭代法也蛮有意思，有时间再写下。