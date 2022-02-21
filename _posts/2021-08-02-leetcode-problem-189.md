# 题目链接
https://leetcode-cn.com/problems/rotate-array/

# 解答过程
## 解法1（k-step）
这个题目比较直接的想法就是把右侧的k个元素暂存下来，右移处理左侧len-k个元素，这样左侧空出k个位置，再把暂存的右侧k个元素填补上即可。

```java
	public void rotate(int[] nums, int k) {
		int len = nums.length;
		if (len == 0) {
			return;
		}

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

## 解法2
一开始我对官方题解的连续reverse方法有点不感冒，但后面仔细想了一下，还是有点意思，具有一定启发性。总结来说，无论是左移还是右移，我们只需要先将数组看作两个子数组，这两个子数组分别reverse，然后再把整个数组reverse，区别无非是左移是把左侧k个元素和剩余元素看作子数组，右移是把右侧k个元素和剩余元素看作子数组。

```java
	public void rotate(int[] nums, int k) {
		int len = nums.length;
		if (len == 0) {
			return;
		}

		k = k % len;
		if (k == 0) {
			return;
		}

		// reverse left len-k elements
		reverse(nums, 0, len - k - 1);

		// reverse right k elements
		reverse(nums, len - k, len - 1);

		// reverse all elements
		reverse(nums, 0, len - 1);
	}

	private void reverse(int[] nums, int left, int right) {
		for (int i = left, j = right; i < j; i++, j--) {
			int tmp = nums[i];
			nums[i] = nums[j];
			nums[j] = tmp;
		}
	}
```