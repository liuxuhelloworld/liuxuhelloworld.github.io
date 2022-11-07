# 题目链接
https://leetcode.cn/problems/median-of-two-sorted-arrays/

# 解答过程
刚入手这个题目时，最直观的解法无非是合并两个有序数组，然后就能很方便地计算出中位数。但是题目里明确要求时间复杂度是O(log(N))，这就有点头疼了。直觉上，既然要求O(log(N))，那么大概率和二分法有关。那么到底怎么二分呢？想了挺久也没有思路。后面看了官方题解才明白怎样把二分法应用到题目中。回头想，如果拿支笔写一写画一画可能也能自己完成数学推导。代码的话，没写二分法的版本，等后面有时间再说吧。这里写了一个简单粗暴的遍历版本，其实和合并有序链表很像。

```java
	public double findMedianSortedArrays(int[] nums1, int[] nums2) {
		int length1 = nums1.length;
		int length2 = nums2.length;

		if (length1 == 1 && length2 == 0) {
			return nums1[0];
		}

		if (length1 == 0 && length2 == 1) {
			return nums2[0];
		}

		int targetCnt1 = (int) Math.ceil((double)(length1+length2) / 2);
		int targetCnt2 = targetCnt1 + 1;

		int target1 = 0, target2 = 0;
		int cnt = 0;

		int i = 0, j = 0;
		while (i < length1 || j < length2) {
			cnt++;

			int min;
			if (i == length1) {
				min = nums2[j++];
			} else if (j == length2) {
				min = nums1[i++];
			} else {
				int val1 = nums1[i];
				int val2 = nums2[j];

				if (val1 <= val2) {
					min = val1;
					i++;
				} else {
					min = val2;
					j++;
				}
			}

			if (cnt == targetCnt1) {
				target1 = min;
			}
			if (cnt == targetCnt2) {
				target2 = min;
				break;
			}
		}

		if ((length1 + length2) % 2 == 1) {
			return target1;
		} else {
			return (double) (target1 + target2) / 2;
		}
	}
```