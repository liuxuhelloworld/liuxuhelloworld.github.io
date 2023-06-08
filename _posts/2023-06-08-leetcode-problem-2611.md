# 题目链接
https://leetcode.cn/problems/mice-and-cheese/

# 解答过程
这个题目乍一看不太好理解，但仔细分析题目要求并不复杂。翻译一下，给定两个等长的数组，现在从数组1的k个位置挑k个元素，从数组2挑其余位置的n-k个元素，期望的是挑出来的元素之和最大，求这个最大和是多少。

对于这种求最值的问题，题目难度设定是中等，我就倾向从动态规划的角度入手。但是，想不出合适的动态规划方程。

那就还是用基于示例的归纳法，尝试从示例问题中寻找解题思路。以示例问题为例，数组1是{1,1,3,3}，数组2是{4,4,1,1}，现在要从数组1中挑2个元素，从数组2中挑其余2个元素，怎么挑能使得挑出来的元素之和最大呢？要不要挑数组1的第1个元素，显然不要，因为数组1的第1个元素是1，数组2的第1个元素是4，如果希望和最大，那么应当尽量倾向数组2的第1个元素被挑出来。同理，数组1的第2个元素也不应该被挑出来。而数组1的第3个和第4个元素应当被挑出来。换句话说，我们应当尽量挑同位置上，数组1比数组2大的那些。如何得到哪些位置上数组1比数组2大呢？我们可以维护一个diff数组，diff每个元素是同位置上数组1和数组2的差值。当我们挑选第1个位置时，我们应当挑diff数组中元素值最大的那个位置。当我们挑选第2个位置时，我们应当挑diff数组中元素值第二大的那个位置。以此类推，题目可解。

```java
	public int miceAndCheese(int[] reward1, int[] reward2, int k) {
		int n = reward1.length;

		int maxCheese;
		if (k <= n/2) {
			maxCheese = maxCheese(reward1, reward2, k);
		} else {
			maxCheese = maxCheese(reward2, reward1, n-k);
		}

		return maxCheese;
	}

	private int maxCheese(int[] reward1, int[] reward2, int k) {
		int n = reward1.length;

		int[] diff = new int[n];
		for (int i = 0; i < n; i++) {
			diff[i] = reward1[i] - reward2[i];
		}

		boolean[] used = new boolean[n];

		for (int i = 0; i < k; i++) {
			int maxDiffIdx = maxDiffIdx(diff);
			used[maxDiffIdx] = true;
			diff[maxDiffIdx] = Integer.MIN_VALUE;
		}

		int maxCheese = 0;
		for (int i = 0; i < n; i++) {
			if (used[i]) {
				maxCheese += reward1[i];
			} else {
				maxCheese += reward2[i];
			}
		}

		return maxCheese;
	}

	private int maxDiffIdx(int[] diff) {
		int maxDiffIdx = 0;
		int maxDiff = diff[0];
		for (int i = 0; i < diff.length; i++) {
			if (diff[i] > maxDiff) {
				maxDiff = diff[i];
				maxDiffIdx = i;
			}
		}

		return maxDiffIdx;
	}
```

代码里每次挑选要遍历diff数组，挑选完置diff数组对应位置为无效值，并且还需要一个布尔数组标识哪些位置被挑除选出来了。为了减少遍历的次数，我还增加了一次如有必要的swap。代码提交以后是可以通过的，不过效率很差。

看了一下官方题解，整体思路还是比较相像的，但是在求元素和的地方有较大不同。细思一下，恍然大悟，确实我的多次循环笨拙而低效了。对于挑选出来的某个位置i，我们的目标是用arr1\[i\]来做加和，arr1\[i\] = arr1\[i\] + arr2\[i\] - arr2\[i\] = arr2\[i\] + arr1\[i\] - arr2\[i\] = arr2\[i\] + diff\[i\]。因此，只需要将数组2的所有元素之和加上diff数组最大的k个元素即可。如果只需要diff数组最大的k个元素，而不需要感知具体位置，那么直接对diff数组做排序就可以了。

```java
	public int miceAndCheese(int[] reward1, int[] reward2, int k) {
		int n = reward1.length;

		int[] diff = new int[n];
		for (int i = 0; i < n; i++) {
			diff[i] = reward1[i] - reward2[i];
		}

		Arrays.sort(diff);

		int maxCheese = 0;
		for (int i = 0; i < n; i++) {
			maxCheese += reward2[i];
		}

		for (int i = 0; i < k; i++) {
			maxCheese += diff[n-1-i];
		}

		return maxCheese;
	}
```

代码提交以后，效率果然好了很多，代码也简洁了很多。

最后多想一下，我上面的较笨拙的解法在某些场景下也有可取之处，比如，题目要求不止计算最大和，还要计算出具体位置。。。