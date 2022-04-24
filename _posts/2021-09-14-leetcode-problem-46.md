# 题目链接
https://leetcode-cn.com/problems/permutations/

# 解答过程
这个题目用回溯法的范式来套，还是比较容易解决的。在回溯的过程中通过一个布尔数组来标记当前哪些元素已经在permutation中，顺序遍历到第一个还不在permutation中的元素，把这个元素入permutation，然后继续回溯。从代码来看，还是比较容易理解和记忆的。

```java
	public List<List<Integer>> permute(int[] nums) {
		List<List<Integer>> result = new ArrayList<>();

		List<Integer> permutation = new ArrayList<>();

		boolean[] tag = new boolean[nums.length];

		backtrack(result, permutation, nums, tag, 0);

		return result;
	}

	private void backtrack(List<List<Integer>> result, List<Integer> permutation, int[] nums, boolean[] tag, int cur) {
		if (cur == nums.length) {
			result.add(new ArrayList<>(permutation));
			return;
		}

		for (int i = 0; i < nums.length; i++) {
			if (tag[i] == false) {
				tag[i] = true;
				permutation.add(nums[i]);
				backtrack(result, permutation, nums, tag, cur+1);
				permutation.remove(permutation.size()-1);
				tag[i] = false;
			}
		}
	}
```

上面这个写法我觉得就可以了，简单清晰易理解。官方题解的写法更简洁，目的是不使用布尔标记数组。仔细体会一下，也不难理解。但是我觉得如果没有相关经验，并不容易想到这种处理方式。

```java
	public List<List<Integer>> permuteV2(int[] nums) {
		List<List<Integer>> result = new ArrayList<>();

		List<Integer> numList = Arrays.stream(nums)
			.boxed()
			.collect(Collectors.toList());

		backtrackV2(result, numList, 0);

		return result;
	}

	private void backtrackV2(List<List<Integer>> result, List<Integer> nums, int cur) {
		if (cur == nums.size()) {
			result.add(new ArrayList<>(nums));
			return;
		}

		for (int i = cur; i < nums.size(); i++) {
			Collections.swap(nums, i, cur);
			backtrackV2(result, nums, cur+1);
			Collections.swap(nums, i, cur);
		}
	}
```