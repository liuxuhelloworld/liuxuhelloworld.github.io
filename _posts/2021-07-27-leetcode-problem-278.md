# 题目链接
https://leetcode-cn.com/problems/first-bad-version/

# 解答过程
这个题目比较容易想到用二分查找，与基本的二分查找的区别在于，基本的二分查找是没有重复值的，这个题目可以看作是对只有两个值的非降序列来确定分界点，其实只要想好循环不变量是什么可以了。

```java
	public int firstBadVersion(int n) {
		assert n >= 1;

		int low = 1, high = n;
		while (low < high) {
			int mid = low + (high - low)/2;
			if (isBadVersion(mid)) {
				high = mid;
			} else {
				low = mid + 1;
			}
		}

		return high;
	}
```

这里的循环不变量就是high始终是第一个错误版本的右边界，或者说，high始终大于或等于第一个错误版本，最终，当low等于high了，已经没有必要再继续，退出循环即可。