# 题目链接
https://leetcode-cn.com/problems/minimum-size-subarray-sum/

# 解答过程
想到的方法就是双循环遍历，外层循环作为左指针，内层循环作为右指针，每次循环找到以当前左指针为起点的满足子数组之和大于等于目标值的最小子数组，然后判断是否更新min。内层循环增加限制条件只处理最多min个元素，因为再多已经没有意义了。如果left为0时的内循环后flag为false，说明所有元素之和都已经不满足题目要求，直接跳出循环即可。

```java
	public int minSubArrayLen(int target, int[] nums) {
		int len = nums.length;

		int min = len;
		boolean flag = false;

		for (int left = 0; left < len; left++) {
			int sum = 0;
			for (int right = left; right < len && right < left + min; right++) {
				sum += nums[right];
				if (sum >= target) {
					if (right - left + 1 <= min) {
						min = right - left + 1;
						flag = true;
						break;
					}
				}
			}

			if (!flag) {
				break;
			}
		}

		return flag ? min : 0;
	}
```

坦白讲，这个代码看着就不漂亮，提交以后结果是通过了，但效率较差。看了下官方题解，官方题解的2有点意思，前缀和这个概念之前没有接触过；官方题解的3看起来就比较漂亮了，这个题目主要考察点应该就是滑动窗口。有时间再写下官方题解的2和3吧。