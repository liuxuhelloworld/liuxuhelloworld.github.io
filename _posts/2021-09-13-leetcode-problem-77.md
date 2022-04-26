# 题目链接
https://leetcode-cn.com/problems/combinations/

# 解答过程
这个题目和Problem39、Problem40、Problem46、Problem47多有相似之处，解法背后的回溯思想是相通的，值得比较并体会。

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