# 题目链接
https://leetcode-cn.com/problems/move-zeroes/

# 解答过程
把所有0挪到数组最右端，其他元素保持原序，通过快慢指针比较容易实现。循环不变量：慢指针左边的元素都是非0的，快指针指向下一个非0元素。这样每一次循环，交换快慢指针的元素，把非0的挪过来，然后继续下一次循环。

```java
	public void moveZeroes(int[] nums) {
		assert nums.length > 0;

		int len = nums.length;

		int slow = 0, fast = 0;
		while (true) {
			while (slow < len && nums[slow] != 0) {
				slow++;
			}
			if (slow == len) {
				break;
			}

			fast = Math.max(fast, slow + 1);
			while (fast < len && nums[fast] == 0) {
				fast++;
			}
			if (fast == len) {
				break;
			}

			int tmp = nums[slow];
			nums[slow] = nums[fast];
			nums[fast] = tmp;

			slow++;
			fast++;
		}
	}
```