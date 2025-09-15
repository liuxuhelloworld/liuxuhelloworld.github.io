# 题目链接
https://leetcode-cn.com/problems/combinations/

# 解答过程
这个题目是经典的组合问题，从n个元素中计算所有k元组，属于适用回溯法的经典问题。相比Problem39、Problem40，这个题目从题目要求到代码实现都更简洁一些。

```java
	public List<List<Integer>> combine(int n, int k) {
		if (k > n) {
			return Collections.emptyList();
		}

		List<List<Integer>> result = new ArrayList<>();
		List<Integer> combination = new ArrayList<>();

		backtrack(result, combination, 1, n, k);

		return result;
	}

	private void backtrack(List<List<Integer>> result, List<Integer> combination, int cur, int n, int k) {
		if (combination.size() == k) {
			result.add(new ArrayList<>(combination));
			return;
		}

		if (cur > n
			|| (n-cur+1) < (k-combination.size())) {
			return;
		}

		combination.add(cur);
		backtrack(result, combination, cur+1, n, k);
		combination.remove(combination.size() - 1);

		backtrack(result, combination, cur+1, n, k);
	}
```