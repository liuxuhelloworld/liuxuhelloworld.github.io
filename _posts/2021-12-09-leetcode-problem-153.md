# 题目链接
https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/

# 解答过程
思考这个题目的时候，脑子里其实是有和官方题解类似的示意图的，两段错开的上升序列，但是在如何二分舍弃时纠结于同时比较nums[mid]和num[left]、nums[right]的大小关系，而没能抽象出只和nums[right]比较即可的思路。nums[mid]和nums[left]的大小关系不足以确认最小值在左侧还是右侧，所以和nums[left]比较没有用。nums[mid]和nums[right]的大小关系就可以确认最小值在左侧还是右侧：
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

这个题目扩展一下，查找最大值也是类似的思路，不赘述。