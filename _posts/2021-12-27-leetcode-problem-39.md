# 题目链接
https://leetcode-cn.com/problems/combination-sum/

# 解答过程
这个题目和Problem40很像，区别在于两点：
（1）Problem39的入参数组无重复元素，Problem40的入参数组可以包含重复元素；
（2）Problem39中每个元素可以使用任意次，Problem40中每个元素最多只能使用1次；
稍微思考一下，就能体会出为什么有这样的设定。因为Problem39中每个元素可以使用任意次，所以即便是允许入参数组包含重复元素，那么也得先排序并过滤掉重复元素，题目里限定无重复元素等于是do you a favor. Problem40允许有重复元素但每个元素至多使用1次，其实就可以理解为Problem39的变种，即可以理解为在Problem39的基础上限定了每个元素最多使用几次。

从代码来看，Problem39比Problem40清爽干净一些，但是思路是一致的，细节不再赘述。

```java
  public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> combination = new ArrayList<>();

    candidates = Arrays.stream(candidates).sorted().toArray();
    backtrack(result, combination, candidates, 0,  target);

    return result;
  }

  private void backtrack(List<List<Integer>> result,
                         List<Integer> combination,
                         int[] candidates,
                         int curIdx,
                         int target) {

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
    backtrack(result, combination, candidates, curIdx+1, target);

    // curIdx included
    combination.add(curVal);
    backtrack(result, combination, candidates, curIdx, target-curVal);
    combination.remove(combination.size()-1);
  }
```