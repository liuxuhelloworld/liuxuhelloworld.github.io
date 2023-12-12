# 题目链接
https://leetcode-cn.com/problems/jump-game-ii/

# 解答过程
## 动态规划
这个题目是挺久以前写的了，看了下和官方题解并不相似，说明当时不是参考官方题解写的，但是真的一点印象也没有了......

这个题目如果从递归的思路来看，还是有比较清晰的解题思路的，可以倒序遍历数组，对于能一步跳到结尾元素的X，问题变为从开头元素跳到X的最小跳数，而这恰好是原问题的更小规模的子问题，由此递归直到开头元素。

一个最优解问题，能通过递归解决，递归过程中有可能重复处理子问题，这些都提示应当使用动态规划。动态规划的关键是定义合适的状态转移方程，这个题目里我们可以定义dp[i]表示从i元素到结尾元素的最小跳数，与上述递归思路类似，同样是倒叙遍历数组，dp[i]的值可以从dp[i+1]到dp[len-1]的值计算而得。

从提交记录看，这种解法的效率很不错。

```java
	public int jump(int[] nums) {
		int len = nums.length;

		if (len == 0 || len == 1) {
			return 0;
		}

		int[] jumps = new int[len]; // jumps[i] means the minimum jumps from nums[i] to the last element
		for (int i = 0; i < len-1; i++) {
			jumps[i] = -1;
		}
		jumps[len-1] = 0;

		for (int i = len-2; i >= 0; i--) {
			jumps[i] = minJumps(jumps, i, nums[i]);
		}

		return jumps[0];
	}

	private int minJumps(int[] jumps, int i, int val) {
		if (val == 0) {
			return -1;
		}

		int min = Integer.MAX_VALUE;
		for (int j = i + 1; j <= i + val && j < jumps.length; j++) {
			if (jumps[j] == -1) {
				continue;
			}
			min = Math.min(min, 1 + jumps[j]);
		}

		if (min == Integer.MAX_VALUE) {
			return -1;
		}

		return min;
	}
```

## 贪婪算法
贪婪算法的思路我是参考了官方题解的，但是坦白讲，官方题解的描述和代码我觉得都不大好理解。我尝试用自己的理解表达一下：贪婪算法嘛，简单说就是每一步都争取最好的那个选择。比方说现在位于nums[0]，那么这第一跳能到达的最远节点的下标是0+nums[0]，记为i。那么问题来了，在[1..i]这些元素中，跳到哪一个最合适？因为对跳数的累加都是1，那么是跳到最远的那个，即下标为i的元素是最优选择吗？不一定。最优选择应该是在能覆盖的这些元素中，挑j+nums[j]的和最大的那个，即跳完这一跳之后下一跳能覆盖最远的那一个。然后，继续循环此过程，直到某一跳能覆盖到最后一个元素了，循环结束。

从代码的角度来说，我也觉得我的代码比官方题解的好理解[狗头]

这个题目可以看作Problem55的后置题目，Problem55是判断有没有可能从起点跳到终点，Problem45是限定可以从起点跳到终点而计算最小跳数。

```java
	public int jump(int[] nums) {
		int len = nums.length;
		if (len == 1) {
			return 0;
		}

		int jump = 0;
		int curIdx = 0;
		int jumpIdx = -1;
		while (true) {
			int curVal = nums[curIdx];
			int canReachIdx = Math.min(curIdx + curVal, len-1);
			if (canReachIdx == len-1) {
				jump++;
				break;
			}

			int maxSum = Integer.MIN_VALUE;
			for (int i = curIdx+1; i <= canReachIdx ; i++) {
				int sum = i + nums[i];
				if (sum > maxSum) {
					maxSum = sum;
					jumpIdx = i;
				}
			}

			jump++;
			curIdx = jumpIdx;
		}

		return jump;
	}
```

# 相似题目
- [Problem11-Container With Most Water](2021-11-03-leetcode-problem-11.md)
- [Problem55-Jump Game](2022-05-09-leetcode-problem-55.md)
- [Problem2178-Maximum Split of Positive Even Integers](2023-07-10-leetcode-problem-2178.md)