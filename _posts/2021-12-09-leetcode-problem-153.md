# 题目链接
https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/

# 解答过程
这个题目和Problem33几乎一样，Problem33是查找特定值，Problem153是查找最小值，都是基于rotation破坏有序性做文章。在Problem33的经验之上，我们可以很容易判断mid节点是在左侧上升段还是右侧上升段。不过Problem153还有一点不同，就是可能rotation后还是全部有序的。正因为这一点，比较nums[mid]和nums[left]的大小关系不足以确认最小值在左侧还是右侧。比如，nums[mid] > nums[left]，如果是Problem33的背景下，那么mid节点一定是在左侧上升段，即最小值在右侧。但是，由于Problem153可能rotation后和原数组一样，那么此时，同样nums[mid] > nums[left]，但是最小值在左侧。因此，和nums[left]比较没有用。相反的，nums[mid]和nums[right]的大小关系就可以确认最小值在左侧还是右侧：
nums[mid] > nums[right]，最小值在右侧，nums[mid]不会是最小值，left=mid+1后继续；
nums[mid] < nums[right]，最小值在左侧，nums[mid]可能是最小值，right=mid后继续。

```java
	public int findMin(int[] nums) {
		int left = 0, right = nums.length - 1;
		while (left < right) {
			int mid = left + (right - left) / 2;
			if (nums[mid] > nums[right]) {
				left = mid + 1;
			} else if (nums[mid] < nums[right]) {
				right = mid;
			}
		}

		return nums[left];
	}
```

# 相似题目
- [Problem4-Median of Two Sorted Arrays](2022-11-07-leetcode-problem-4.md)
- [Problem33-Search in Rotated Sorted Array](2022-01-04-leetcode-problem-33.md)
- [Problem81-Search in Rotated Sorted Array](2023-11-09-leetcode-problem-81.md)