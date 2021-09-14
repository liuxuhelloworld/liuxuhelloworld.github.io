# 题目链接
https://leetcode-cn.com/problems/permutations/

# 解答过程
这个题目也是以前写的了，记不清当时有没有看官方题解。单纯从代码看，还是比较容易理解的。

## 解法1
```java
	public List<List<Integer>> permute(int[] nums) {
		List<List<Integer>> result = new ArrayList<>();

		List<Integer> numList = Arrays.stream(nums)
			.boxed()
			.collect(Collectors.toList());

		backtrack(result, numList, 0);

		return result;
	}

	private void backtrack(List<List<Integer>> result, List<Integer> nums, int cur) {
		if (cur == nums.size()) {
			result.add(new ArrayList<>(nums));
			return;
		}

		for (int i = cur; i < nums.size(); i++) {
			Collections.swap(nums, i, cur);
			backtrack(result, nums, cur+1);
			Collections.swap(nums, i, cur);
		}
	}
```

## 解法2
```java
	public List<List<Integer>> permuteV2(int[] nums) {
		List<List<Integer>> result = new ArrayList<>();

		List<Integer> permutation = new ArrayList<>();

		boolean[] tag = new boolean[nums.length];

		backtrackV2(result, permutation, nums, tag, 0);

		return result;
	}

	private void backtrackV2(List<List<Integer>> result, List<Integer> permutation, int[] nums, boolean[] tag, int cur) {
		if (cur == nums.length) {
			result.add(new ArrayList<>(permutation));
			return;
		}

		for (int i = 0; i < nums.length; i++) {
			if (tag[i] == false) {
				tag[i] = true;
				permutation.add(nums[i]);
				backtrackV2(result, permutation, nums, tag, cur+1);
				permutation.remove(permutation.size()-1);
				tag[i] = false;
			}
		}
	}
```