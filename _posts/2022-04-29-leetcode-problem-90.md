# 题目链接
https://leetcode-cn.com/problems/subsets-ii/

# 解答过程
Problem90是Problem78的变种，就像Problem40是Problem39的变种一样，回溯的思想都是一致的。Problem78限定入参无重复元素，计算子集时，等效于每个元素可以选择0次或1次；Problem90限定入参可以包含重复元素，计算子集时，等效于每个元素可以选择0,1,2..,X次，其中X是入参中这个元素重复出现的次数。

```java
	public List<List<Integer>> subsetsWithDup(int[] nums) {
		List<List<Integer>> subsets = new ArrayList<>();
		List<Integer> subset = new ArrayList<>();

		Arrays.sort(nums);

		List<int[]> numsCount = new ArrayList<>();
		Arrays.stream(nums).forEach(num -> {
			if (numsCount.size() > 0 && numsCount.get(numsCount.size() - 1)[0] == num) {
				numsCount.get(numsCount.size() - 1)[1] += 1;
			} else {
				numsCount.add(new int[] {num, 1});
			}
		});

		backtrack(subsets, subset, numsCount, 0);

		return subsets;
	}

	private void backtrack(List<List<Integer>> subsets, List<Integer> subset, List<int[]> numsCount, int idx) {
		if (idx == numsCount.size()) {
			subsets.add(new ArrayList<>(subset));
			return;
		}

		int[] cur = numsCount.get(idx);
		int num = cur[0], count = cur[1];
		for (int i = 0; i <= count; i++) {
			for (int j = 0; j < i; j++) {
				subset.add(num);
			}

			backtrack(subsets, subset, numsCount, idx + 1);

			for (int j = 0; j < i; j++) {
				subset.remove(subset.size() - 1);
			}
		}
	}
```

瞅了一眼官方题解，我觉得还不如我的思路好理解，-_-