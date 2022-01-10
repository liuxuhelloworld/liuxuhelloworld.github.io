# 题目链接
https://leetcode-cn.com/problems/interval-list-intersections/

# 解答过程
这个题目的解答相当不顺利。其实思路还是比较容易想到的，有点像合并有序链表，两个指针i和j分别挨个指向firstList和secondList，对于当前指向的两个interval，判断它们是否相交，如果相交，把相交的部分记录下来，无论是否相交，i和j当中某一个需要前进。哪一个需要前进呢？当前指向的两个interval中比较偏左的那一个需要前进。

这个思路虽然没有严格的数据证明，但是直觉上是正确的，后面看了官方题解，思路也是差不多的。

提交以后报错解答错误，一开始一个错误是我在判断i和j哪一个需要前进时判断错了，对于当前指向的两个interval，我用它们的左端点来判断那一个偏左，这样是不准确的，应该使用右端点。

改正了这里后提交，还是报错，报错的输入实例是很长的list，很难手动debug了。只好看了官方题解，和官方题解的思路是差不多的，官方题解的代码更多地使用了库函数，更简洁一些，但是为什么我的提交报错呢？那只能有一个原因了，就是我判断两个区间是否相关的方法有问题。

```java
	private boolean intersect(int[] first, int[] second) {
		return (first[0] - second[1]) * (first[1] - second[0]) <= 0;
	}
```

这里我自作聪明地写了这么个方法，直觉上也觉得是对的，但显然不是。。。换成官方题解的判断方法后果然没问题了。

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