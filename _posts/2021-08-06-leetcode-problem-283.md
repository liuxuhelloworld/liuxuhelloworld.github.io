# 题目链接
https://leetcode-cn.com/problems/move-zeroes/

# 解答过程
把所有0挪到数组最右端，其他元素保持原序，通过快慢指针比较容易实现。慢指针指向下一个要覆盖的位置，快指针遍历数组找非0元素，找到以后写到慢指针对应的位置，同时更新慢指针。快指针遍历完后跳出循环，再把慢指针开始往后的元素都赋值0即可。

```java
	public void moveZeroes(int[] nums) {
		int len = nums.length;

		int slow = 0;
		for (int fast = 0; fast < len; fast++) {
			if (nums[fast] != 0) {
				nums[slow] = nums[fast];
				slow++;
			}
		}

		for (int i = slow; i < len; i++) {
			nums[i] = 0;
		}
	}
```