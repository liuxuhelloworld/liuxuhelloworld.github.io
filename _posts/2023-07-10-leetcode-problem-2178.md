# 题目链接
https://leetcode.cn/problems/maximum-split-of-positive-even-integers/

# 解答过程
这个题目乍一看有点唬人，翻译一下就是把一个数做偶数之和分解，要求不能有重复元素，可能有多种分解方式，求包含元素最多的分解方式包含的元素个数。显然，如果目标值是奇数，那么解不存在。如果是偶数，那么怎么计算包含元素最多的分解方式包含的元素个数呢？

从题目要求到题目示例，我很直接就想起来回溯法...，说起来也逻辑也很顺，回溯法找出所有可能的分解方式，然后取包含元素最多的那种就可以了。吭哧吭哧开始写，

```java
	public List<Long> maximumEvenSplit(long finalSum) {
		if (finalSum % 2 != 0) {
			return Collections.emptyList();
		}

		List<List<Long>> splits = new ArrayList<>();
		List<Long> split = new ArrayList<>();
		backtrack(splits, split, 0, finalSum);

		if (splits.isEmpty()) {
			return Collections.emptyList();
		}

		int maxSize = 0;
		int maxIdx = -1;
		for (int i = 0; i < splits.size(); i++) {
			split = splits.get(i);
			if (split.size() > maxSize) {
				maxSize = split.size();
				maxIdx = i;
			}
		}

		return splits.get(maxIdx);
	}

	private void backtrack(List<List<Long>> splits, List<Long> split, long last, long remain) {
		if (remain == 0) {
			splits.add(new ArrayList<>(split));
			return;
		}

		if (last > remain) {
			return;
		}

		long next = last + 2;
		while (next <= remain) {
			split.add(next);
			backtrack(splits, split, next, remain - next);
			split.remove(split.size() - 1);

			next += 2;
		}
	}
```

到这里还挺洋洋自得呢，虽然挺长时间没写回溯题目了，但还比较手顺。结果提交以后不通过，超时。

其实本地自测的时候就隐约觉得不对劲，比如，一个并不大的目标值28，可能的分解方式已经很多了，但其实第一个回溯出的结果就是满足题目要求的。我想在回溯过程中加一个escape，回溯出第一个结果就结束，但总觉得这种处理方式怪怪的。无奈看了一眼官方题解，额，果然犯了方向性的错误，这个题目更适合于贪婪算法。

想一下，其实也很好理解，为了使分解后的元素数尽量多，那么就应该尽量用比较小的数，所以就应该2、4、6、8、...这样挨个累加，到最接近目标值时，把与目标值最后的那点差距加到最后一个元素上即可。为什么这样做是正确的呢？反正法。设定上述方式得到的分解方式包含的元素个数为n，如果n不是最优结果，那么必然存在包含元素数为n+1的分解方式，根据我们的分解逻辑，n+1的这种分解方式必然大于目标值了。因此得证。

```java
	public List<Long> maximumEvenSplit(long finalSum) {
		if (finalSum % 2 != 0) {
			return Collections.emptyList();
		}

		List<Long> split = new ArrayList<>();
		long val = 2;
		long remain = finalSum;
		while (val <= remain) {
			split.add(val);
			remain -= val;
			val += 2;
		}

		int size = split.size();
		long last = split.get(size - 1);
		split.set(size - 1, last + remain);

		return split;
	}
```