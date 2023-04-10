# 题目链接
https://leetcode.cn/problems/previous-permutation-with-one-swap/

# 解答过程
这个题目从题目本身的要求来看，并不难理解，交换任意两个位置的元素称为一次swap，现在需要找到一组swap，swap后的数组，以lexicographically的方式来比较，小于当前数组，但是要越大越好。lexicographically嘛，简单来说就是挨个元素比较，以第一个不相等的元素的大小关系为整个数组的大小关系。

题目的要求理解了，那么如何找到一组swap，能满足题目要求呢？首先比较容易确认的一点是，我们需要找到的是一对逆序的元素，比如，(...,3,...,2)，swap后是(...,2,...,3)，这样才能保证swap后得到的数组小于原数组。

那么下一个问题就是哪一对逆序元素swap后得到的数组在小于当前数组的限制下能尽可能大？答案是这一对逆序元素要尽可能靠右。因为swap一对逆序元素是把一个较小的值挪到了高位，那么要想让swap后的数组尽可能大，就要尽量不影响较高位的元素，即要尽量swap靠右的逆序元素对。

那么如何找到最靠右的逆序元素对？其实并不难想到，逆序遍历原数组，每次遍历比较当前值与前值的大小关系，当碰到第一个前值大于当前值时，即找到了最靠右的逆序元素对，暂时称为(prevVal, curVal)。是这样吗？乍一看好像没什么问题，但其实不是。我们确实找到了最靠右的逆序元素对的左元素，即prevVal，但是最靠右的逆序元素对的右元素可能还能更靠右，因为在已遍历的元素中可能存在大于curVal但是小于prevVal的元素，比如，(...,9,3,4,5)，当我们找到(9,3)时，(9,3)其实并不是我们需要的答案，而是要在已遍历的元素中再去找到大于3而小于9的最大的那个元素，这样才满足题目要求，即(9,...,5)。

```java
	public int[] prevPermOpt1(int[] arr) {
		int i = arr.length - 1;
		while (i > 0) {
			if (arr[i - 1] > arr[i]) {
				break;
			}
			i--;
		}

		if (i == 0) {
			return arr;
		}

		int leftIdx = i - 1;
		int rightIdx = i;

		int curVal = arr[i];
		int prevVal = arr[i - 1];

		int maxVal = curVal;
		int j = i + 1;
		while (j < arr.length) {
			if (arr[j] > curVal && arr[j] < prevVal) {
				if (arr[j] > maxVal) {
					rightIdx = j;
					maxVal = arr[j];
				}
			}
			j++;
		}

		swap(arr, leftIdx, rightIdx);

		return arr;
	}

	private void swap(int[] arr, int left, int right) {
		int tmp = arr[left];
		arr[left] = arr[right];
		arr[right] = tmp;
	}
```

这个题目前前后后花了将近一个半小时才做出来，每次都是提交以后不通过，根据bad case再去修补，直到最后才整理出合理的思路，归根结底还是翻译问题并抽象出计算思路的能力不足。

看了一下官方题解，表述不太一样，但思路其实是差不多的。