# 题目链接
https://leetcode-cn.com/problems/combination-sum/

# 解答过程
这个题目应该是比较经典的DFS题目了，我觉得应该全文背诵-_-

```java
  public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> combination = new ArrayList<>();

    candidates = Arrays.stream(candidates).sorted().toArray();
    dfs(result, combination, candidates, 0,  target);

    return result;
  }

  private void dfs(List<List<Integer>> result, List<Integer> combination, int[] candidates,
      int curIdx, int target) {

    if (curIdx == candidates.length) {
      return;
    }

    if (target == 0) {
      result.add(new ArrayList<>(combination)); // pay attention here
      return;
    }

    int curVal = candidates[curIdx];
    if (curVal > target) {
      return;
    }

    // curIdx excluded
    dfs(result, combination, candidates, curIdx+1, target);

    // curIdx included
    combination.add(curVal);
    dfs(result, combination, candidates, curIdx, target-curVal);
    combination.remove(combination.size()-1);
  }
```