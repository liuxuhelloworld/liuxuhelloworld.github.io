# 问题977 有序数组的平方 ★☆☆☆☆
https://leetcode-cn.com/problems/squares-of-a-sorted-array/

如何以O(N)的时间复杂度来解决这个问题呢？我一开始的思路是先遍历到第一个正数的位置，然后从该位置向两边遍历。看了下官方题解才明白，直接从两端向中间遍历就可以嘛。很好的体现了双指针的概念。

```java
	public int[] sortedSquares(int[] nums) {
		assert nums.length > 0;

		int len = nums.length;

		int[] squares = new int[len];

		int i = 0, j = len - 1, k = len - 1;
		while (i <= j) {
			if (Math.abs(nums[i]) >= Math.abs(nums[j])) {
				squares[k--] = nums[i] * nums[i];
				i++;
			} else {
				squares[k--] = nums[j] * nums[j];
				j--;
			}
		}

		return squares;
	}
```

# 问题986 区间列表的交集 ★★☆☆☆
https://leetcode-cn.com/problems/interval-list-intersections/

这个题目的解题思路还是比较容易想到的，有点像合并有序链表，两个指针i和j分别挨个指向firstList和secondList，对于当前指向的两个interval，判断它们是否相交，如果相交，把相交的部分记录下来，无论是否相交，i和j当中某一个需要前进。哪一个需要前进呢？当前指向的两个interval中比较偏左的那一个需要前进。

```java
	public int[][] intervalIntersection(int[][] firstList, int[][] secondList) {
		int m = firstList.length;
		int n = secondList.length;

		List<List<Integer>> intersectionList = new ArrayList<>();

		int i = 0, j = 0;
		while (i < m && j < n) {
			int[] curFirst = firstList[i];
			int[] curSecond = secondList[j];

			List<Integer> curIntersection = new ArrayList<>(2);

			int low = Math.max(curFirst[0], curSecond[0]);
			int high = Math.min(curFirst[1], curSecond[1]);
			if (low <= high) {
				curIntersection.add(low);
				curIntersection.add(high);

				intersectionList.add(curIntersection);
			}

			if (curFirst[1] < curSecond[1]) {
				i++;
			} else {
				j++;
			}
		}

		int size = intersectionList.size();

		if (size == 0) {
			return new int[][] {};
		}

		int[][] intersections = new int[size][2];
		for (i = 0; i < intersections.length; i++) {
			List<Integer> cur = intersectionList.get(i);
			intersections[i][0] = cur.get(0);
			intersections[i][1] = cur.get(1);
		}

		return intersections;
	}
```