# 题目链接
https://leetcode-cn.com/problems/rotate-array/

# 解答过程
这个题的关键就是当数据量较大时，如何尽量高效地完成操作。一个元素一个元素的ont-step右移显然不合适，k-step就差不多了。

```java
	public void rotate(int[] nums, int k) {
		assert nums.length > 0;
		assert k >= 0;

		int len = nums.length;

		k = k % len;
		if (k == 0) {
			return;
		}

		int[] tmp = new int[k];
		for (int i = 0; i < k; i++) {
			tmp[i] = nums[len - k + i];
		}

		for (int i = len - k - 1; i >= 0; i--) {
			nums[i + k] = nums[i];
		}

		for (int i = 0; i < k; i++) {
			nums[i] = tmp[i];
		}
	}
```

我觉得这个题目写到这就可以了，思路比较清晰也容易理解，官方题解里的连续reverse方法感觉有点tricky.