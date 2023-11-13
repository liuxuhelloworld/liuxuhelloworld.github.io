# 题目链接
https://leetcode-cn.com/problems/first-bad-version/

# 解答过程
把这个题目和Problem34有一定相似度，都是二分查找的基本变种，相对比较简单，在二分查找的基础上稍作修改即可，不赘述。

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

# 相似题目
- [Problem34-Find First and Last Position of Elements in Sorted Array](2021-11-25-leetcode-problem-34.md)