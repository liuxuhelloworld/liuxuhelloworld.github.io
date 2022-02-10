# 题目链接
https://leetcode-cn.com/problems/first-bad-version/

# 解答过程
这个题目比较简单，在二分查找的基础上稍作即可，不赘述。

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