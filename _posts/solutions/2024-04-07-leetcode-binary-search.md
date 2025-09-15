<a id="problem_4"></a>
# 问题4 寻找两个正序数组的中位数 ★★★☆☆
https://leetcode.cn/problems/median-of-two-sorted-arrays/

这个题目本身并不难理解，给定两个有序数组，求这两个有序数组一起构成的集合的中位数。如果是给定一个有序数组，我们很容易得到这个有序数组的中位数，只需要根据数组长度计算中位数的下标，然后直接读取对应的数据即可。那么显然，这个题目我们可以先合并两个有序数组，然后就可以很方便地计算中位数了。合并有序数组并不复杂，和合并有序链表的思路类似，代码还简单很多。合并有序数组这种解法，显然时间复杂度是O(N)的，空间复杂度也是O(N)。一种优化方式是不做真正的合并，而是在模拟合并的过程中计数，当遍历到中位数的位置时，模拟合并就结束了。这种优化方式的时间复杂度一样还是O(N)的，主要是空间复杂度可以降低到O(1)。

考虑到这个题目的难度是困难，题目里也明确要求时间复杂度为O(log(N))，那么显然上面两种解法都不满足题目要求。那么如何才能达到O(log(N))的时间复杂度呢？有序数组，O(log(N))，如果有经验或者好直觉的话，可以想到，应该从二分法的角度入手。我们可以先翻译一下我们的问题：给定两个有序数组，我们需要以O(log(N))的时间复杂度得到这两个有序数组的第K小元素。既然是从二分法入手，那么我们可以再翻译一下我们的问题：如何快速排除N=K/2个元素。既然两个数组都是有序的，那么我们可以分别取这两个数组的前N个元素，那么显然它们一起构成了这两个有序数组的最小2N个元素（2N\<=K，因为N=K/2是整数除法）。换句话说，第K小元素要么是这2N个元素的最大值，要么不在这2N个元素中。我们可以比较这两个长度为N的子数组的最大元素，对于较小的那个，连同它以及它前面这N个元素，我们可以直接排除，因为第K小的元素不会在这个子数组中。这样，我们不就是快速排除了N=K/2个元素么？当然，这里面还有一些边界问题需要注意，比如两个数组可能一个已经为空、比如取第1小的元素等

```java
	public double findMedianSortedArrays(int[] nums1, int[] nums2) {
		int len1 = nums1.length;
		int len2 = nums2.length;

		if ((len1 + len2) % 2 == 0) {
			int cnt1 = (len1 + len2) / 2;
			int cnt2 = (len1 + len2) / 2 + 1;
			int val1 = findKthNumberOfTwoSortedArrays(nums1, nums2, cnt1);
			int val2 = findKthNumberOfTwoSortedArrays(nums1, nums2, cnt2);
			return (double)(val1 + val2) / 2;
		} else {
			int cnt = (len1 + len2) / 2 + 1;
			int val = findKthNumberOfTwoSortedArrays(nums1, nums2, cnt);
			return (double) val;
		}
	}

	private int findKthNumberOfTwoSortedArrays(int[] nums1, int[] nums2, int k) {
		int idx1 = 0, idx2 = 0;
		while (true) {
			if (idx1 >= nums1.length) {
				return nums2[idx2 + k - 1];
			}

			if (idx2 >= nums2.length) {
				return nums1[idx1 + k - 1];
			}

			if (k == 1) {
				return Math.min(nums1[idx1], nums2[idx2]);
			}

			int n = k/2;
			int newIdx1 = Math.min(idx1 + n - 1, nums1.length - 1);
			int newIdx2 = Math.min(idx2 + n - 1, nums2.length - 1);
			int val1 = nums1[newIdx1];
			int val2 = nums2[newIdx2];
			if (val1 <= val2) {
				k = k - (newIdx1 - idx1 + 1);
				idx1 = newIdx1 + 1;
			} else {
				k = k - (newIdx2 - idx2 + 1);
				idx2 = newIdx2 + 1;
			}
		}
	}
```

<a id="problem_33"></a>
# 问题33 搜索旋转排序数组 ★★☆☆☆
https://leetcode-cn.com/problems/search-in-rotated-sorted-array/

这个题目是经典的二分查找的变种，而题目要求又是O(log(N))的时间复杂度，那么直觉上应该还是从二分查找入手。对于二分查找的题目，重点就是二分后如何舍弃一半。经典的二分查找算法中，由于单调递增性，通过比较mid元素和target就可以确认舍弃哪一半。这个题目中，由于rotation破坏了单调递增性，比较mid元素和target不足以确认该舍弃哪一半。关键点就在于单调递增性。那么如何获得单调递增性呢？通过比较mid元素和第一个元素就可以确认\[0..mid\]和\[mid..n-1\]哪一部分是具备单调递增性的，由此再和target比较来确认该舍弃哪一半。

这个题目和[问题4](#problem_4)有一定相似之处，都是基于有序数组做二分法的文章，问题4是把单个数组的有序性拆分为两个有序数组，问题33是通过rotation把单个数组的有序性变成两段有序子数组，问题的关键点都是基于破坏后的有序性做二分法。

```java
  public int search(int[] nums, int target) {
    if (nums.length == 0) {
      return -1;
    }

    int left = 0, right = nums.length - 1;

    while (left <= right) {
      int mid = left + (right - left)/2;
      if (nums[mid] == target) {
        return mid;
      }

      if (nums[0] <= nums[mid]) {
        // left part is ordered
        if (target >= nums[0] && target < nums[mid]) {
          right = mid - 1;
        } else {
          left = mid + 1;
        }
      } else {
        // right part is ordered
        if (target > nums[mid] && target <= nums[nums.length-1]) {
          left = mid + 1;
        } else {
          right = mid - 1;
        }
      }
    }

    return -1;
  }
```

<a id="problem_34"></a>
# 问题34 在排序数组中查找元素的第一个和最后一个位置 ★☆☆☆☆
https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/

这个题目属于二分查找的变种，直觉上就是连续二分查找。看了官方题解，我觉得还不如这种处理容易理解。

```java
  public int[] searchRange(int[] nums, int target) {
    if (nums.length == 0) {
      return new int[] {-1, -1};
    }

    int leftest = -1, rightest = -1;
    int index = binarySearch(nums, 0, nums.length-1, target);
    if (index == -1) {
      return new int[] {-1, -1};
    } else {
      leftest = rightest = index;
    }

    while ((index = binarySearch(nums, 0, leftest-1, target)) != -1) {
      leftest = index;
    }

    while ((index = binarySearch(nums, rightest+1, nums.length-1, target)) != -1) {
      rightest = index;
    }

    return new int[] {leftest, rightest};
  }

  private int binarySearch(int[] nums, int left, int right, int target) {
    if (left > right) {
      return -1;
    }

    if ((left >= 0 && target < nums[left])
        || (right <= nums.length-1 && target > nums[right])) {
      return -1;
    }

    while (left <= right) {
      int mid = left + (right - left)/2;
      if (nums[mid] == target) {
        return mid;
      } else if (nums[mid] > target) {
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    }

    return -1;
  }
```

<a id="problem_81"></a>
# 问题81 搜索旋转排序数组 ★★☆☆☆
https://leetcode.cn/problems/search-in-rotated-sorted-array-ii/

这个题目和[问题33](#problem_33)几乎一样，差异点有两个：
1. 问题33里数组元素是distinct的，即没有重复值，而问题81里数组元素可能有重复值；
2. 问题33里rotation的位置是从第二个元素开始，即rotation后必然是两段，而问题81里rotation的位置可能是第一个元素，即rotation后可能和原数组是一样的。

因为上述差异点，问题81就比问题33难一些，尤其是当nums[mid]和nums[0]相等时，我们很难判断出当前哪个部分具备有序性。但是话说回来，当nums[mid] > nums[0]或者nums[mid] < nums[0]时，和问题33类似，我们可以判断出nums[left..mid]或者nums[mid..right]是不是有序的。而对于nums[mid] == nums[0]这种情况，最简单粗暴的做法就是同时更新left和right，然后继续下一层循环。

```java
	public boolean search(int[] nums, int target) {
		int left = 0, right = nums.length-1;

		while (left <= right) {
			int mid = left + (right-left)/2;

			if (nums[left] == target
				|| nums[right] == target
				|| nums[mid] == target) {
				return true;
			}

			if (nums[mid] > nums[0]) {
				// num[left..mid] is no-decreasing sorted
				if (target > nums[left] && target < nums[mid]) {
					right = mid - 1;
				} else {
					left = mid + 1;
				}
			} else if (nums[mid] < nums[0]) {
				// num[mid..right] is non-decreasing sorted
				if (target > nums[mid] && target < nums[right]) {
					left = mid + 1;
				} else {
					right = mid - 1;
				}
			} else {
				left++;
				right--;
			}

		}

		return false;
	}
```

<a id="problem_153"></a>
# 问题153 寻找旋转排序数组中的最小值 ★★☆☆☆
https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/

这个题目和[问题33](#problem_33)几乎一样，问题33是查找特定值，问题153是查找最小值，都是基于rotation破坏有序性做文章。在问题33的经验之上，我们可以很容易判断mid节点是在左侧上升段还是右侧上升段。不过问题153还有一点不同，就是可能rotation后还是全部有序的。正因为这一点，比较nums[mid]和nums[left]的大小关系不足以确认最小值在左侧还是右侧。比如，nums[mid] > nums[left]，如果是问题33的背景下，那么mid节点一定是在左侧上升段，即最小值在右侧。但是，由于问题153可能rotation后和原数组一样，那么此时，同样nums[mid] > nums[left]，但是最小值在左侧。因此，和nums[left]比较没有用。相反的，nums[mid]和nums[right]的大小关系就可以确认最小值在左侧还是右侧：
nums[mid] > nums[right]，最小值在右侧，nums[mid]不会是最小值，left=mid+1后继续；
nums[mid] < nums[right]，最小值在左侧，nums[mid]可能是最小值，right=mid后继续。

```java
	public int findMin(int[] nums) {
		int left = 0, right = nums.length - 1;
		while (left < right) {
			int mid = left + (right - left) / 2;
			if (nums[mid] > nums[right]) {
				left = mid + 1;
			} else if (nums[mid] < nums[right]) {
				right = mid;
			}
		}

		return nums[left];
	}
```

# 问题704 二分查找 ☆☆☆☆☆
https://leetcode-cn.com/problems/binary-search/

```java
	public int search(int[] nums, int target) {
		assert nums.length > 0;

		int left = 0, right = nums.length - 1;

		while (left <= right) {
			int mid = left + (right - left) / 2;
			if (nums[mid] == target) {
				return mid;
			} else if (nums[mid] > target) {
				right = mid - 1;
			} else {
				left = mid + 1;
			}
		}

		return -1;
	}
```

# 问题35 搜索插入位置 ★☆☆☆☆
https://leetcode-cn.com/problems/search-insert-position/

这个题目比较简单了，属于二分查找的微小变种，在二分查找的基础上增加一个插入位置下标的计算而已。稍微想一下两个连续元素的二分查找过程就可以确认最后返回right+1即可。

```java
	public int searchInsert(int[] nums, int target) {
		assert nums.length > 0;

		int left = 0, right = nums.length - 1;
		while (left <= right) {
			int mid = left + (right - left) / 2;
			if (nums[mid] == target) {
				return mid;
			} else if (nums[mid] > target) {
				right = mid - 1;
			} else {
				left = mid + 1;
			}
		}

		return right + 1;
	}
```

# 问题278 第一个错误的版本 ★☆☆☆☆
https://leetcode-cn.com/problems/first-bad-version/

这个题目是二分查找的变种，相比问题35稍微难一点点，在二分查找的基础上稍作修改即可。值得注意的地方是，这里二分查找的循环条件是**low < high**，而不是**low <= high**，注意体会为什么是这样。其实也很好理解，把握住循环不变量是什么就可以了：在循环过程中，low是非错误的版本，high始终是错误的版本。最终会达到low和high相邻，此时high恰好是第一个错误的版本。

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

# 问题74 搜索二维矩阵 ★☆☆☆☆
https://leetcode-cn.com/problems/search-a-2d-matrix/

这个题目其实就是把一个非递减序列的结构由一维数组换成了二维数组，问题的本质依然是二分查找，先按第一列做二分查找确认target所在行，再按行做二分查找。

```java
	public boolean searchMatrix(int[][] matrix, int target) {
		int row = binarySearch(matrix, target);
		if (row == -1) {
			return false;
		}

		return binarySearch(matrix[row], target);
	}

	private int binarySearch(int[][] matrix, int target) {
		int m = matrix.length;
		int n = matrix[0].length;

		int top = 0, down = m-1;
		while (top <= down) {
			int mid = top + (down-top)/2;
			if (target >= matrix[mid][0] && target <= matrix[mid][n-1]) {
				return mid;
			} else if (target < matrix[mid][0]) {
				down = mid - 1;
			} else if (target > matrix[mid][n-1]) {
				top = mid + 1;
			}
		}

		return -1;
	}

	private boolean binarySearch(int[] row, int target) {
		int left = 0, right = row.length-1;
		while (left <= right) {
			int mid = left + (right-left)/2;
			if (target == row[mid]) {
				return true;
			} else if (target < row[mid]){
				right = mid - 1;
			} else if (target > row[mid]) {
				left = mid + 1;
			}
		}

		return false;
	}
```

# 问题162 寻找峰值 ★☆☆☆☆
https://leetcode-cn.com/problems/find-peak-element/

这个题目的关键是二分后如何舍弃某一半：
- 当前二分点恰好是一个peek点，那么直接返回即可
- 当前二分点不是peek点，那么当前二分点可能处于三种状态：小于左邻大于右邻，大于左邻小于右邻，小于左邻小于右邻。我们只需要向较大的邻点靠近就可以了，即舍弃较小的邻点部分，如果同时小于左邻和右邻，随便选一个就可以了。为什么继续二分较大的邻点部分一定可以找到peek点呢？因为根据题目隐含要求，nums[-1]和nums[nums.length]默认是极小值，所以从当前二分点后继续处理是向上走，而边界点往外是向下走，这当中必然有一个peek点。

```java
	public int findPeakElement(int[] nums) {
		int len = nums.length;
		if (len == 1) {
			return 0;
		}

		int left = 0, right = len - 1;
		while (left <= right) {
			int mid = left + (right - left) / 2;
			if (isPeak(nums, mid)) {
				return mid;
			} else if (nums[mid] < nums[mid+1]) {
				left = mid + 1;
			} else {
				right = mid - 1;
			}
		}

		return -1; // logically unreachable statement
	}

	private boolean isPeak(int[] nums, int i) {
		int len = nums.length;
		if (len == 1) {
			return true;
		}

		if (i == 0) {
			return nums[i] > nums[i+1];
		}

		if (i == len - 1) {
			return nums[i] > nums[i-1];
		}

		return nums[i] > nums[i-1] && nums[i] > nums[i+1];
	}
```