# 题目链接
https://leetcode.cn/problems/maximum-fruits-harvested-after-at-most-k-steps/

# 解答过程
题目要求并不难理解，从某个startPos开始，每一步可以向左走或者向右走，最多可以走k步，凡是经过的点的水果就可以收割，求最多能收割多少水果。

一开始我的想法是用类似宽度优先遍历BFS的方式去暴力计算，模拟k步，每一步计算当前可能到达的点是哪些，以及每一个可能到达的点的累加水果数，最终取走完k步后能到达的点中累加水果数最大的那一个。乍一想好像还挺有道理，草草写了一版后验证，果然不行。稍作分析，可以发现，上面的思路中在首次访问到某个点时就会把该点上的水果收割，而其他路径再走到这个点时已经没有水果可收割，即不同路径之间有干扰。换句话说，要想用类似的暴力方式计算，需要维护每一条路径，并累加路径上的水果数，最终得到水果数最大的路径。由于路径数是指数级上升的，并不可取。

没有什么思路的情况下，我一般会从示例问题入手，尝试基于具体的、小规模的问题推理解法。比如示例问题中从startPos=5开始走4步，那么显然，你有以下这些选择：
- 往左走4步，此时位于点1
- 往左走3步，再往右1步，此时位于点3
- 往左走2步，再往右2步，此时回到点5
- 往左走1步，再往右3步，此时位于点7
- 往右走1步，再往左3步，此时位于点3
- 往右走2步，再往左2步，此时回到点5
- 往右走3步，再往左1步，此时位于点7
- 往右走4步，此时位于点9

发现规律了吗？走k步可以做类似拆分，而每一种走法下能收割的水果数由以下三部分组成：
- startPos点的水果数
- startPos以左能到达的最远点内累加的水果数
- startPos以右能到达的最远点内累加的水果数
  
startPos以左能到达的每个点内累加的水果数恰恰是可以预加工出来的，startPos以右能到达的每个点内累加的水果数也是一样可以预加工出来的。

```java
	public int maxTotalFruits(int[][] fruits, int startPos, int k) {
		int length = fruits.length;
		int maxIdx = Math.max(fruits[length - 1][0], startPos);
		int[] fruitArr = new int[maxIdx + 1];
		for (int i = 0; i < length; i++) {
			fruitArr[fruits[i][0]] = fruits[i][1];
		}

		if (k == 0) {
			return fruitArr[startPos];
		}

		// left[i] represents harvested fruits number when go left i step, startPos not included
		int maxLeftSteps = Math.min(startPos, k);
		int[] leftFruits = new int[maxLeftSteps + 1];
		for (int i = 1; i < leftFruits.length; i++) {
			leftFruits[i] = leftFruits[i-1] + fruitArr[startPos - i];
		}

		// right[i] represents harvested fruits number when go right i step, startPos not included
		int maxRightSteps = Math.min(maxIdx - startPos, k);
		int[] rightFruits = new int[maxRightSteps + 1];
		for (int i = 1; i < rightFruits.length; i++) {
			rightFruits[i] = rightFruits[i-1] + fruitArr[startPos + i];
		}

		int maxFruits = 0;

		int startPosFruits = fruitArr[startPos];

		for (int leftStep = 0; leftStep <= maxLeftSteps; leftStep++) {
			int leftStepFruits = leftFruits[leftStep];

			int rightStep = k - (leftStep * 2);
			if (rightStep <= 0) {
				rightStep = 0;
			} else if (rightStep > maxRightSteps) {
				rightStep = maxRightSteps;
			}
			int rightStepFruits = rightFruits[rightStep];

			if (startPosFruits + leftStepFruits + rightStepFruits > maxFruits) {
				maxFruits = startPosFruits + leftStepFruits + rightStepFruits;
			}
		}

		for (int rightStep = 0; rightStep <= maxRightSteps; rightStep++) {
			int rightStepFruits = rightFruits[rightStep];

			int leftStep = k - (rightStep * 2);
			if (leftStep <= 0) {
				leftStep = 0;
			} else if (leftStep > maxLeftSteps) {
				leftStep = maxLeftSteps;
			}
			int leftStepFruits = leftFruits[leftStep];

			if (startPosFruits + leftStepFruits + rightStepFruits > maxFruits) {
				maxFruits = startPosFruits + leftStepFruits + rightStepFruits;
			}
		}

		return maxFruits;
	}
```

这个题目刚入手时，还是有点害怕的，因为很少做hard难度的题目。。。看了下官方题解，两种解法都似懂非懂，只能说官方题解更专业更有深度，但我觉得我的解法更好理解。