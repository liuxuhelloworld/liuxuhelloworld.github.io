# 题目链接
https://leetcode.cn/problems/median-of-two-sorted-arrays/

# 解答过程
这个题目本身并不难理解，给定两个有序数组，求这两个有序数组一起构成的集合的中位数。如果是给定一个有序数组，我们很容易得到这个有序数组的中位数，只需要根据数组长度计算中位数的下标，然后直接读取对应的数据即可。那么显然，这个题目我们可以先合并两个有序数组，然后就可以很方便地计算中位数了。合并有序数组并不复杂，和合并有序链表的思路类似，代码还简单很多。合并有序数组这种解法，显然时间复杂度是O(N)的，空间复杂度也是O(N)。一种优化方式是不做真正的合并，而是在模拟合并的过程中计数，当遍历到中位数的位置时，模拟合并就结束了。这种优化方式的时间复杂度一样还是O(N)的，主要是空间复杂度可以降低到O(1)。

考虑到这个题目的难度是困难，题目里也明确要求时间复杂度为O(log(N))，那么显然上面两种解法都不满足题目要求。那么如何才能达到O(log(N))的时间复杂度呢？有序数组，O(log(N))，如果有经验或者好直觉的话，可以想到，应该从二分法的角度入手。我们可以先翻译一下我们的问题：给定两个有序数组，我们需要以O(log(N))的时间复杂度得到这两个有序数组的第K小元素。既然是从二分法入手，那么我们可以再翻译一下我们的问题：如何快速排除N=K/2个元素。既然两个数组都是有序的，那么我们可以分别取这两个数组的前N个元素，那么显然它们一起构成了这两个有序数组的最小2N个元素（2N\<=K，因为N=K/2是整数除法）。换句话说，第K小元素要么是这2N个元素的最大值，要么不在这2N个元素中。我们可以比较这两个长度为N的子数组的最大元素，对于较小的那个，连同它以及它前面这N个元素，我们可以直接排除，因为第K小的元素不会在这个子数组中。这样，我们不就是快速排除了N=K/2个元素么？当然，这里面还有一些边界问题需要注意，比如两个数组可能一个已经为空、比如取第1小的元素等

这个题目是在看了官方题解以后写出来的，甚至不是当时就写出来的，而是再三思考加尝试以后才写出来。另外，我觉得相比官方题解的代码，我这里计算new index的写法更好理解[狗头]

```java
	public double findMedianSortedArrays(int[] nums1, int[] nums2) {
		int len1 = nums1.length;
		int len2 = nums2.length;

		if ((len1 + len2) % 2 == 0) {
			int cnt1 = (len1 + len2) / 2;
			int cnt2 = (len1 + len2) / 2 + 1;
			int val1 = findKthNumberOfTwoSortedArrays(nums1, nums2, cnt1);
			int val2 = findKthNumberOfTwoSortedArrays(nums1, nums2, cnt2);
			return (double)(val1 + val2) / 2;
		} else {
			int cnt = (len1 + len2) / 2 + 1;
			int val = findKthNumberOfTwoSortedArrays(nums1, nums2, cnt);
			return (double) val;
		}
	}

	private int findKthNumberOfTwoSortedArrays(int[] nums1, int[] nums2, int k) {
		int idx1 = 0, idx2 = 0;
		while (true) {
			if (idx1 >= nums1.length) {
				return nums2[idx2 + k - 1];
			}

			if (idx2 >= nums2.length) {
				return nums1[idx1 + k - 1];
			}

			if (k == 1) {
				return Math.min(nums1[idx1], nums2[idx2]);
			}

			int n = k/2;
			int newIdx1 = Math.min(idx1 + n - 1, nums1.length - 1);
			int newIdx2 = Math.min(idx2 + n - 1, nums2.length - 1);
			int val1 = nums1[newIdx1];
			int val2 = nums2[newIdx2];
			if (val1 <= val2) {
				k = k - (newIdx1 - idx1 + 1);
				idx1 = newIdx1 + 1;
			} else {
				k = k - (newIdx2 - idx2 + 1);
				idx2 = newIdx2 + 1;
			}
		}
	}
```

# 相似题目
- [Problem33-Search in Rotated Sorted Array](2022-01-04-leetcode-problem-33.md)
- [Problem81-Search in Rotated Sorted Array](2023-11-09-leetcode-problem-81.md)
- [Problem153-Find Minimum in Rotated Sorted Array](2021-12-09-leetcode-problem-153.md)
- [Problem34-Find First and Last Position of Elements in Sorted Array](2021-11-25-leetcode-problem-34.md)