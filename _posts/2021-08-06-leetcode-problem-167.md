# 题目链接
https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/

# 解答过程
题目明确数组non-decreasing，并且答案唯一，利用这两点限制，通过左右指针向中间靠拢，根据当前左右指针指向元素的sum决定是更新左指针还是右指针。

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