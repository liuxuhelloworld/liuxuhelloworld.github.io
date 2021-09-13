# 题目链接
https://leetcode-cn.com/problems/combinations/

# 解答过程
这个题目是以前写的了，印象中当时也是参考官方题解写的。有了一些回溯的代码经验后，还是比较容易理解和掌握的。

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