# 问题11 盛最多水的容器 ★☆☆☆☆
https://leetcode-cn.com/problems/container-with-most-water/

这个题目，从入手角度来说，我首先想的是暴力解法，稍作思考就可以知道，暴力解法很简单，双重循环就可以了，O(N^2)时间复杂度。考虑到题目难度是中等，显然暴力解法是不符合考察要求的。那么如何更快的计算出最大值呢？

对于最值问题，我总是第一反应是尝试使用动态规划，但是对于这个题目，思考了下，设计不出合适的状态转移方程，换句话说，并不适合动态规划解法。

官方题解把这个题目的解法归为双指针，我更愿意看作某种贪婪算法。从首尾两根挡板开始，首先计算出当前的最大值，即(j-i)*min(height[i], height[j])，其中i是第一个挡板，j是最后一个挡板。然后我们开始从首尾向中间靠近，舍弃height[i]和height[j]中较小的那根挡板，然后计算一个新的最大值，并决定是否更新全局最大值。为什么舍弃小的那根挡板是正确的呢？因为从首尾向中间靠近的过程中，(j-i)是不断减小的，既然底部的边是不断减小的，那么我们至少要尝试增大挡板的高度，所以舍弃height[i]和height[j]中较小的那根挡板至少保留了得到更大值的可能性，因此舍弃小的那边循环处理就能得到最终答案。

```java
    public int maxArea(int[] height) {
        int len = height.length;

        int containerWidth;
        int containerHeight;
        int maxArea = 0;

        int i = 0, j = len-1;
        while(i < j) {
            containerWidth = j - i;
            containerHeight = Math.min(height[i], height[j]);
            maxArea = Math.max(maxArea, containerWidth*containerHeight);

            if (height[i] < height[j]) {
                i++;
            } else {
                j--;
            }
        }

        return maxArea;
    }
```

<a id="problem_45"></a>
# 问题45 跳跃游戏 ★☆☆☆☆
https://leetcode-cn.com/problems/jump-game-ii/

贪婪算法的思路我是参考了官方题解的，但是坦白讲，官方题解的描述和代码我觉得都不大好理解。我尝试用自己的理解表达一下：贪婪算法嘛，简单说就是每一步都争取最好的那个选择。比方说现在位于nums[0]，那么这第一跳能到达的最远节点的下标是0+nums[0]，记为i。那么问题来了，在[1..i]这些元素中，跳到哪一个最合适？因为对跳数的累加都是1，那么是跳到最远的那个，即下标为i的元素是最优选择吗？不一定。最优选择应该是在能覆盖的这些元素中，挑j+nums[j]的和最大的那个，即跳完这一跳之后下一跳能覆盖最远的那一个。然后，继续循环此过程，直到某一跳能覆盖到最后一个元素了，循环结束。

```java
	public int jump(int[] nums) {
		int len = nums.length;
		if (len == 1) {
			return 0;
		}

		int jump = 0;
		int curIdx = 0;
		int jumpIdx = -1;
		while (true) {
			int curVal = nums[curIdx];
			int canReachIdx = Math.min(curIdx + curVal, len-1);
			if (canReachIdx == len-1) {
				jump++;
				break;
			}

			int maxSum = Integer.MIN_VALUE;
			for (int i = curIdx+1; i <= canReachIdx ; i++) {
				int sum = i + nums[i];
				if (sum > maxSum) {
					maxSum = sum;
					jumpIdx = i;
				}
			}

			jump++;
			curIdx = jumpIdx;
		}

		return jump;
	}
```

<a id="problem_55"></a>
# 问题55 跳跃游戏 ★☆☆☆☆
https://leetcode.cn/problems/jump-game/

我的思路是这样的，分析题目，这个题目的关键在于0元素点，如果除结尾元素外，没有0元素点，那么肯定是能从开始元素跳到结尾元素的。所以可以倒序循环找0元素点，如果找到了，那么需要判断在它前面是否有某个元素可以跳过这个0元素点，如果没有，那说明跳不到结尾元素了，循环结束；如果有，那么继续上述循环处理即可。看了眼官方题解，官方题解代码简洁很多，但思路我觉得是差不多的，都可以看作贪婪算法的思路。

这个题目可以看作[问题45](#problem_45)的前置题目，问题55是判断有没有可能从起点跳到终点，问题45是限定可以从起点跳到终点而计算最小跳数。

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

# 问题2611 老鼠和奶酪 ★☆☆☆☆
https://leetcode.cn/problems/mice-and-cheese/

这个题目乍一看不太好理解，但仔细分析题目要求并不复杂。翻译一下，给定两个等长的数组，现在从数组1的k个位置挑k个元素，从数组2挑其余位置的n-k个元素，期望的是挑出来的元素之和最大，求这个最大和是多少。

对于这种求最值的问题，题目难度设定是中等，我就倾向从动态规划的角度入手。但是，想不出合适的动态规划方程。

那就还是用基于示例的归纳法，尝试从示例问题中寻找解题思路。以示例问题为例，数组1是{1,1,3,3}，数组2是{4,4,1,1}，现在要从数组1中挑2个元素，从数组2中挑其余2个元素，怎么挑能使得挑出来的元素之和最大呢？要不要挑数组1的第1个元素，显然不要，因为数组1的第1个元素是1，数组2的第1个元素是4，如果希望和最大，那么应当尽量倾向数组2的第1个元素被挑出来。同理，数组1的第2个元素也不应该被挑出来。而数组1的第3个和第4个元素应当被挑出来。换句话说，我们应当尽量挑同位置上，数组1比数组2大的那些。如何得到哪些位置上数组1比数组2大呢？我们可以维护一个diff数组，diff每个元素是同位置上数组1和数组2的差值。当我们挑选第1个位置时，我们应当挑diff数组中元素值最大的那个位置。当我们挑选第2个位置时，我们应当挑diff数组中元素值第二大的那个位置。以此类推，题目可解。

```java
	public int miceAndCheese(int[] reward1, int[] reward2, int k) {
		int n = reward1.length;

		int maxCheese;
		if (k <= n/2) {
			maxCheese = maxCheese(reward1, reward2, k);
		} else {
			maxCheese = maxCheese(reward2, reward1, n-k);
		}

		return maxCheese;
	}

	private int maxCheese(int[] reward1, int[] reward2, int k) {
		int n = reward1.length;

		int[] diff = new int[n];
		for (int i = 0; i < n; i++) {
			diff[i] = reward1[i] - reward2[i];
		}

		boolean[] used = new boolean[n];

		for (int i = 0; i < k; i++) {
			int maxDiffIdx = maxDiffIdx(diff);
			used[maxDiffIdx] = true;
			diff[maxDiffIdx] = Integer.MIN_VALUE;
		}

		int maxCheese = 0;
		for (int i = 0; i < n; i++) {
			if (used[i]) {
				maxCheese += reward1[i];
			} else {
				maxCheese += reward2[i];
			}
		}

		return maxCheese;
	}

	private int maxDiffIdx(int[] diff) {
		int maxDiffIdx = 0;
		int maxDiff = diff[0];
		for (int i = 0; i < diff.length; i++) {
			if (diff[i] > maxDiff) {
				maxDiff = diff[i];
				maxDiffIdx = i;
			}
		}

		return maxDiffIdx;
	}
```

代码里每次挑选要遍历diff数组，挑选完置diff数组对应位置为无效值，并且还需要一个布尔数组标识哪些位置被挑除选出来了。为了减少遍历的次数，我还增加了一次如有必要的swap。代码提交以后是可以通过的，不过效率很差。

看了一下官方题解，整体思路还是比较相像的，但是在求元素和的地方有较大不同。细思一下，恍然大悟，确实我的多次循环笨拙而低效了。对于挑选出来的某个位置i，我们的目标是用arr1\[i\]来做加和，arr1\[i\] = arr1\[i\] + arr2\[i\] - arr2\[i\] = arr2\[i\] + arr1\[i\] - arr2\[i\] = arr2\[i\] + diff\[i\]。因此，只需要将数组2的所有元素之和加上diff数组最大的k个元素即可。如果只需要diff数组最大的k个元素，而不需要感知具体位置，那么直接对diff数组做排序就可以了。

```java
	public int miceAndCheese(int[] reward1, int[] reward2, int k) {
		int n = reward1.length;

		int[] diff = new int[n];
		for (int i = 0; i < n; i++) {
			diff[i] = reward1[i] - reward2[i];
		}

		Arrays.sort(diff);

		int maxCheese = 0;
		for (int i = 0; i < n; i++) {
			maxCheese += reward2[i];
		}

		for (int i = 0; i < k; i++) {
			maxCheese += diff[n-1-i];
		}

		return maxCheese;
	}
```

代码提交以后，效率果然好了很多，代码也简洁了很多。

最后多想一下，我上面的较笨拙的解法在某些场景下也有可取之处，比如，题目要求不止计算最大和，还要计算出具体位置。。。

# 问题767 重构字符串 ★★☆☆☆
https://leetcode.cn/problems/reorganize-string/

刚入手这个题目时，有点想当然了，直觉上的思路是先计数字符个数，然后按个数由大到小，往重组后的字符串里塞，当找不到可塞的位置时，认为是bad case. 提交以后不通过，很快就发现了问题在哪。一个简单的例子，假设个数最少的字符是两个，其他字符都很好的塞到了重组后字符串的最左端，现在还剩下两个位置，那么就会处理成bad case, 但其实这并不是bad case. 所以，该怎么解决这个问题呢？其实这个时候，我们只需要把上一轮的字符往后塞就可以了。OK，当前轮处理发现进行不下去了，需要回到上一轮重新调整，这不就是回溯么？所以，如果一开始就从回溯的角度考虑，思路就很容易打开了，甚至不需要计数字符个数，就是遍历原字符串，挨个往重组后的字符串里塞，塞不下去了就往前回溯，处理到原字符串末尾时说明已经得到了一个符合条件的重组字符串。按照这个思路写了一版之后提交，发现还是没通过，超时了......。看了一下超时case，应该是bad case导致的递归深度太大。在最外层加了一个可行性判断，回溯时也优化了下，改为每次取剩余计数最多的字符优先处理。提交通过，只是效率还是一般。

看了一下官方题解，官方题解里通过堆来维护字符个数，然后每次取个数top2的两个字符往结果字符串里塞这个处理方法还是有一定启发性的，尤其是每次取个数top2的两个字符，稍微思考下就能明白确实是简洁有效的处理方法，可以理解成贪婪算法。

参考官方题解，把回溯法的版本又改了一下，确实没有必要按回溯处理。不过我没有使用堆，从计数数组里也很容易计算出当前个数top2的字符。

```java
	public String reorganizeString(String s) {
		char[] origin = s.toCharArray();

		int maxCount = 0, allCount = origin.length;

		int[] count = new int[26];
		for (int i = 0; i < origin.length; i++) {
			int idx = origin[i] - 'a';
			count[idx]++;

			if (count[idx] > maxCount) {
				maxCount = count[idx];
			}
		}

		if (allCount - maxCount < maxCount - 1) {
			return "";
		}

		StringBuilder builder = new StringBuilder();

		int maxCountIdx, secondMaxCountIdx;
		while (true) {
			int[] top2CountCharsIdx = getTop2CountCharsIdx(count);

			maxCountIdx = top2CountCharsIdx[0];
			secondMaxCountIdx = top2CountCharsIdx[1];
			if (maxCountIdx == -1 && secondMaxCountIdx == -1) {
				break;
			}

			if (maxCountIdx != -1) {
				builder.append((char)('a' + maxCountIdx));
				count[maxCountIdx]--;
			}
			if (secondMaxCountIdx != -1) {
				builder.append((char)('a' + secondMaxCountIdx));
				count[secondMaxCountIdx]--;
			}
		}

		return builder.toString();
	}

	private int[] getTop2CountCharsIdx(int[] count) {
		int maxCount = 0, maxCountIdx = -1, secondMaxCount = 0, secondMaxCountIdx = -1;
		for (int i = 0; i < count.length; i++) {
			if (count[i] > maxCount) {
				secondMaxCount = maxCount;
				secondMaxCountIdx = maxCountIdx;

				maxCount = count[i];
				maxCountIdx = i;
			} else if (count[i] > secondMaxCount) {
				secondMaxCount = count[i];
				secondMaxCountIdx = i;
			}
		}

		return new int[] {maxCountIdx, secondMaxCountIdx};
	}
```

# 问题846 一手顺子 ★★☆☆☆
https://leetcode-cn.com/problems/hand-of-straights/

首先，根据题目要求，如果hand包含的元素值个数不能被groupSize整除，那么显然不能满足分组要求。那么接下来该如何判断是否能满足分组要求呢？我的直觉是先计数，统计每个元素出现的次数。然后我只能想到比较暴力的解法就是按元素值从小到大（因此计数元素个数的map需要使用TreeMap），以groupSize为单元，扣减计数。如果处理到某个元素时当前groupSize大小的单元内不足扣减，或者全部处理完后还有元素的计数大于0，那么就不能满足分组要求，否则就可以满足分组要求。按照这个思路写了一版代码，提交以后也能通过。

看了下官方题解，整体思路上有一些共通之处，但是官方题解里先排序，然后以排序后的hand为遍历对象的处理显然更好，代码更简洁更优雅一些，更能体现贪婪算法的思想。这个题目蛮有意思的，很好的体现了贪婪算法的思想。

```java
	public boolean isNStraightHand(int[] hand, int groupSize) {
		int len = hand.length;
		if (len % groupSize != 0) {
			return false;
		}

		Arrays.sort(hand);

		Map<Integer, Integer> cnt = new HashMap<>();
		for (int val : hand) {
			cnt.put(val, cnt.getOrDefault(val, 0) + 1);
		}

		for (int val : hand) {
			boolean contains = cnt.containsKey(val);
			if (!contains) {
				continue;
			}

			for (int i = 0; i < groupSize; i++) {
				int neededVal = val + i;
				contains = cnt.containsKey(neededVal);
				if (!contains) {
					return false;
				} else {
					int valCnt = cnt.get(neededVal);
					--valCnt;
					if (valCnt > 0) {
						cnt.put(neededVal, valCnt);
					} else {
						cnt.remove(neededVal);
					}
				}
			}
		}

		return true;
	}
```

# 问题1053 交换一次的先前排列 ★★☆☆☆
https://leetcode.cn/problems/previous-permutation-with-one-swap/

这个题目从题目本身的要求来看，并不难理解，交换任意两个位置的元素称为一次swap，现在需要找到一组swap，swap后的数组，以lexicographically的方式来比较，小于当前数组，但是要越大越好。lexicographically嘛，简单来说就是挨个元素比较，以第一个不相等的元素的大小关系为整个数组的大小关系。

题目的要求理解了，那么如何找到一组swap，能满足题目要求呢？首先比较容易确认的一点是，我们需要找到的是一对逆序的元素，比如，(...,3,...,2)，swap后是(...,2,...,3)，这样才能保证swap后得到的数组小于原数组。

那么下一个问题就是哪一对逆序元素swap后得到的数组在小于当前数组的限制下能尽可能大？答案是这一对逆序元素要尽可能靠右。因为swap一对逆序元素是把一个较小的值挪到了高位，那么要想让swap后的数组尽可能大，就要尽量不影响较高位的元素，即要尽量swap靠右的逆序元素对。

那么如何找到最靠右的逆序元素对？其实并不难想到，逆序遍历原数组，每次遍历比较当前值与前值的大小关系，当碰到第一个前值大于当前值时，即找到了最靠右的逆序元素对，暂时称为(prevVal, curVal)。是这样吗？乍一看好像没什么问题，但其实不是。我们确实找到了最靠右的逆序元素对的左元素，即prevVal，但是最靠右的逆序元素对的右元素可能还能更靠右，因为在已遍历的元素中可能存在大于curVal但是小于prevVal的元素，比如，(...,9,3,4,5)，当我们找到(9,3)时，(9,3)其实并不是我们需要的答案，而是要在已遍历的元素中再去找到大于3而小于9的最大的那个元素，这样才满足题目要求，即(9,...,5)。

看了一下官方题解，表述不太一样，但思路其实是差不多的。

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