# 题目链接
https://leetcode-cn.com/problems/find-peak-element/

# 解答过程
这个题目的关键是二分后如何舍弃某一半：
- 当前二分点恰好是一个peek点，那么直接返回即可
- 当前二分点不是peek点，那么当前二分点可能处于三种状态：小于左邻大于右邻，大于左邻小于右邻，小于左邻小于右邻。我们只需要向较大的邻点靠近就可以了，即舍弃较小的邻点部分，如果同时小于左邻和右邻，随便选一个就可以了。为什么继续二分较大的邻点部分一定可以找到peek点呢？因为根据题目隐含要求，nums[-1]和nums[nums.length]默认是极小值，所以从当前二分点后继续处理是向上走，而边界点往外是向下走，这当中必然有一个peek点。

```java
	public int findPeakElement(int[] nums) {
		int len = nums.length;
		if (len == 1) {
			return 0;
		}

		int left = 0, right = len - 1;
		while (left <= right) {
			int mid = left + (right - left) / 2;
			if (isPeak(nums, mid)) {
				return mid;
			} else if (nums[mid] < nums[mid+1]) {
				left = mid + 1;
			} else {
				right = mid - 1;
			}
		}

		return -1; // logically unreachable statement
	}

	private boolean isPeak(int[] nums, int i) {
		int len = nums.length;
		if (len == 1) {
			return true;
		}

		if (i == 0) {
			return nums[i] > nums[i+1];
		}

		if (i == len - 1) {
			return nums[i] > nums[i-1];
		}

		return nums[i] > nums[i-1] && nums[i] > nums[i+1];
	}
```

看了下官方题解，思路是差不多的，但我觉得我的代码更好理解。