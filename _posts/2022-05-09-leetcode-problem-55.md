# 题目链接
https://leetcode.cn/problems/jump-game/

# 解答过程
这个题目也是挺久以前写的了。我的思路是这样的，分析题目，这个题目的关键在于0元素点，如果除结尾元素外，没有0元素点，那么肯定是能从开始元素跳到结尾元素的。所以可以倒序循环找0元素点，如果找到了，那么需要判断在它前面是否有某个元素可以跳过这个0元素点，如果没有，那说明跳不到结尾元素了，循环结束；如果有，那么继续上述循环处理即可。看了眼官方题解，官方题解代码简洁很多，但思路我觉得是差不多的，都可以看作贪心算法的思路。

```java
	public boolean canJump(int[] nums) {
		int len = nums.length;

		int idx = len - 1;
		int lastZeroIdx;
		while ((lastZeroIdx = lastZero(nums, idx)) != -1) {
			idx = canJumpZeroLastIdx(nums, lastZeroIdx);
			if (idx == -1) {
				return false;
			}
		}

		return true;
	}

	private int lastZero(int[] nums, int idx) {
		for (int i = idx - 1; i >= 0; i--) {
			if (nums[i] == 0) {
				return i;
			}
		}

		return -1;
	}

	private int canJumpZeroLastIdx(int[] nums, int zeroIdx) {
		for (int i = zeroIdx - 1; i >= 0; i--) {
			if (nums[i] > (zeroIdx - i)) {
				return i;
			}
		}

		return -1;
	}
```