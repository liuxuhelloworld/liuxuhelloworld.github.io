# 题目链接
https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/

# 解答过程
这个题目和Problem1很相似，最大的区别在于这个题目的数组是非减有序的。Problem1的解法用到这个题目上没有问题，但那样等于没有利用起来树组有序的特性。为了利用起来树组有序的特性，固定一个元素后，对其右的元素做二分查找，是比较符合直觉的。通过左右指针向中间靠拢，根据当前左右指针指向元素的sum决定是更新左指针还是右指针，这种方式更简洁，逻辑上的正确性需要一定的推理。

```java
	public int[] twoSum(int[] numbers, int target) {
		int left = 0, right = numbers.length - 1;
		while (left < right) {
			int sum = numbers[left] + numbers[right];
			if (sum == target) {
				break;
			} else if (sum > target) {
				right--;
			} else {
				left++;
			}
		}
		return new int[] {left+1, right+1};
	}
```

# 相似题目
- [Problem1-Two Sum](2022-10-12-leetcode-problem-1.md)
- [Problem15-Three Sum](2021-11-22-leetcode-problem-15.md)
- [Problem18-Four Sum](2021-11-24-leetcode-problem-18.md)
- [Problem16-Three Sum Closest](2021-11-23-leetcode-problem-16.md)