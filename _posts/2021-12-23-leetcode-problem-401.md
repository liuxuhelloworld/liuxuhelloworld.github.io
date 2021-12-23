# 题目链接
https://leetcode-cn.com/problems/binary-watch/

# 解答过程
这个题目从回溯法的角度来考虑的话，可以抽象为从一个包含N个元素的数组中，挑选M个数出来，要求它们的和满足一定限制，返回所有可能的挑选对应的和的集合。

```java
	public List<String> readBinaryWatch(int turnedOn) {
		int[] hourVal = new int[] {8, 4, 2, 1};
		int[] minuteVal = new int[] {32, 16, 8, 4, 2, 1};

		List<String> result = new ArrayList<>();

		for (int hourTurned = 0; hourTurned <= turnedOn; hourTurned++) {
			int minuteTurned = turnedOn - hourTurned;

			Set<Integer> hourResult = new HashSet<>();
			boolean[] hourSelected = new boolean[hourVal.length];
			backtrack(hourResult, hourSelected, hourVal, hourTurned, 11);

			Set<Integer> minuteResult = new HashSet<>();
			boolean[] minuteSelected = new boolean[minuteVal.length];
			backtrack(minuteResult, minuteSelected, minuteVal, minuteTurned, 59);

			for (Integer hour : hourResult) {
				for (Integer minute : minuteResult) {
					String time = hour + ":" + (minute >= 10 ? "" + minute : "0" + minute);
					result.add(time);
				}
			}
		}

		return result;
	}

	private void backtrack(Set<Integer> result, boolean[] selected, int[] nums, int remain, int max) {
		if (remain > nums.length) {
			return;
		}

		if (remain == 0) {
			int sum = sum(nums, selected);
			if (sum <= max) {
				result.add(sum);
			}
			return;
		}

		for (int i = 0; i < selected.length; i++) {
			if (!selected[i]) {
				selected[i] = true;
				backtrack(result, selected, nums, remain - 1, max);
				selected[i] = false;
			}
		}
	}

	private int sum(int[] nums, boolean[] selected) {
		int sum = 0;
		for (int i = 0; i < nums.length; i++) {
			if (selected[i]) {
				sum += nums[i];
			}
		}

		return sum;
	}
```

然后我就吭哧吭哧写了以上代码，正确性没有问题，但是效率较差，尤其是回溯过程中有大量重复运算，本来目标是计算挑选子集的和，是顺序无关的，但这里等于是按顺序有关进行的运算。

然后看了一眼官方题解，额，怪不得难度是简单，抽时间再写下二进制位的解法。