# 题目链接
https://leetcode.cn/problems/search-in-rotated-sorted-array-ii/

# 解答过程
这个题目和Problem33几乎一样，差异点有两个：
1. Problem33里数组元素是distinct的，即没有重复值，而Problem81里数组元素可能有重复值；
2. Problem33里rotation的位置是从第二个元素开始，即rotation后必然是两段，而Problem81里rotation的位置可能是第一个元素，即rotation后可能和原数组是一样的。

因为上述差异点，Problem81就比Problem33难一些，尤其是当nums[mid]和nums[0]相等时，我们很难判断出当前哪个部分具备有序性。但是话说回来，当nums[mid] > nums[0]或者nums[mid] < nums[0]时，和Problem33类似，我们可以判断出nums[left..mid]或者nums[mid..right]是不是有序的。而对于nums[mid] == nums[0]这种情况，最简单粗暴的做法就是同时更新left和right，然后继续下一层循环。

```java
	public boolean search(int[] nums, int target) {
		int left = 0, right = nums.length-1;

		while (left <= right) {
			int mid = left + (right-left)/2;

			if (nums[left] == target
				|| nums[right] == target
				|| nums[mid] == target) {
				return true;
			}

			if (nums[mid] > nums[0]) {
				// num[left..mid] is no-decreasing sorted
				if (target > nums[left] && target < nums[mid]) {
					right = mid - 1;
				} else {
					left = mid + 1;
				}
			} else if (nums[mid] < nums[0]) {
				// num[mid..right] is non-decreasing sorted
				if (target > nums[mid] && target < nums[right]) {
					left = mid + 1;
				} else {
					right = mid - 1;
				}
			} else {
				left++;
				right--;
			}

		}

		return false;
	}
```

# 相似题目
- [Problem4-Median of Two Sorted Arrays](2022-11-07-leetcode-problem-4.md)
- [Problem33-Search in Rotated Sorted Array](2022-01-04-leetcode-problem-33.md)
- [Problem153-Find Minimum in Rotated Sorted Array](2021-12-09-leetcode-problem-153.md)