<a id="problem_1"></a>
# 问题1 两数之和 ☆☆☆☆☆
https://leetcode.cn/problems/two-sum/

对于这个题目，双循环的暴力解法是很显然的一个思路，时间复杂度O(N^2)。要想降低时间复杂度，就需要以更快的速度判断出某个值是否存在，不仅是否存在，如果存在还需要知道这个值在原数组中的下标，由此，需要使用哈希表。

```java
	public int[] twoSum(int[] nums, int target) {
		Map<Integer, Integer> map = new HashMap<>(nums.length);

		for (int i = 0; i < nums.length; i++) {
			int minus = target - nums[i];
			if (map.containsKey(minus)) {
				return new int[] {map.get(minus), i};
			}

			map.put(nums[i], i);
		}

		return new int[0];
	}
```

<a id="problem_167"></a>
# 问题167 两数之和 ☆☆☆☆☆
https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/

# 解答过程
这个题目和[问题1](#problem_1)很相似，最大的区别在于这个题目的数组是非减有序的。Problem1的解法用到这个题目上没有问题，但那样等于没有利用起来树组有序的特性。为了利用起来树组有序的特性，固定一个元素后，对其右的元素做二分查找，是比较符合直觉的。通过左右指针向中间靠拢，根据当前左右指针指向元素的sum决定是更新左指针还是右指针，这种方式更简洁，逻辑上的正确性需要一定的推理。

```java
	public int[] twoSum(int[] numbers, int target) {
		int left = 0, right = numbers.length - 1;
		while (left < right) {
			int sum = numbers[left] + numbers[right];
			if (sum == target) {
				break;
			} else if (sum > target) {
				right--;
			} else {
				left++;
			}
		}
		return new int[] {left+1, right+1};
	}
```

<a id="problem_15"></a>
# 问题15 三数之和 ★★☆☆☆
https://leetcode-cn.com/problems/3sum/

题目本身很好理解，给定一个数组，找出所有相加之和为0的三元组，注意还需要对三元组去重。

如果从暴力解法入手，直接三重循环，是很容易找出所有满足条件的三元组的，但一方面这种做法时间效率较差，另一方面如何对三元组去重呢？

把这个题目和[问题1](#problem_1)、[问题167](#problem_167)放到一起来看，我们可以先排序，排序以后循环处理，每次循环固定一个元素，从剩余元素中按问题167中的双指针解法确定另两个元素。另外，通过判断固定的数是不是已经处理过来去重。这样一个解题思路，如果基于问题1、问题167来思考，倒也不显得很难。但是，如果没有问题1、问题167打底，思路上又有一定难度。

```java
	public List<List<Integer>> threeSum(int[] nums) {
		int len = nums.length;
		if (len == 0) {
			return Collections.emptyList();
		}

		Arrays.sort(nums);

		List<List<Integer>> ret = new ArrayList<>();
		for (int i = 0; i < len; i++) {
			if (i > 0 && nums[i] == nums[i-1]) {
				continue;
			}

			int target = 0 - nums[i];
			int j = i + 1, k = len - 1;
			while (j < k) {
				if (j > i+1 && nums[j] == nums[j-1]) {
					j++;
					continue;
				}

				int curSum = nums[j] + nums[k];
				if (curSum > target) {
					k--;
				} else if (curSum < target) {
					j++;
				} else {
					ret.add(Arrays.asList(nums[i], nums[j], nums[k]));
					j++;
					k--;
				}
			}
		}

		return ret;
	}
```

<a id="problem_16"></a>
# 问题16 最接近的三数之和 ★★☆☆☆
https://leetcode-cn.com/problems/3sum-closest/

这个题目和[问题15](#problem_15)几乎一样，只是这里是计算最接近目标值的三元组的和是多少。从直觉上，我觉得这种最值问题给人的感觉更难。对于这个题目，解题思路和问题15是一致的，不赘述。还是想提一下的是，基于问题1 --> 问题167 --> 问题15 --> 问题16这样一个顺序层层递进下来，解题思路就显得”水到渠成“，否则，死记硬背也容易忘记。

```java
	public int threeSumClosest(int[] nums, int target) {
		int len = nums.length;
		if (len == 0) {
			throw new RuntimeException("bad input");
		}

		Arrays.sort(nums);

		int ret = 0;
		int minDiff = Integer.MAX_VALUE;
		for (int i = 0; i < len; i++) {
			if (i > 0 && nums[i] == nums[i - 1]) {
				continue;
			}

			int j = i+1, k = len-1;
			while (j < k) {
				if (j > i + 1 && nums[j] == nums[j - 1]) {
					j++;
					continue;
				}

				int curSum = nums[i] + nums[j] + nums[k];

				int curDiff = Math.abs(curSum - target);
				if (curDiff < minDiff) {
					ret = curSum;
					minDiff = curDiff;
				}

				if (curSum > target) {
					k--;
				} else if (curSum < target) {
					j++;
				} else {
					return curSum;
				}
			}
		}

		return ret;
	}
```

<a id="problem_18"></a>
# 问题18 四数之和 ★★☆☆☆
https://leetcode-cn.com/problems/4sum/

这个题目和[问题15](#problem_15)非常像了，那么类似的，解题的思路也是一致的，只不过比问题15多一层循环。值得注意的是，如果基于问题1 --> 问题167 --> 问题15 --> 问题18这样一个顺序来层层递进，那么解题的思路是有脉络可循的，但是如果上来就是直接写这样一个题目，我感觉会比较懵，不容易有清晰的思路。

```java
	public List<List<Integer>> fourSum(int[] nums, int target) {
		int len = nums.length;
		if (len < 4) {
			return Collections.emptyList();
		}

		List<List<Integer>> ret = new ArrayList<>();

		Arrays.sort(nums);

		for (int i = 0; i < len; i++) {
			if (i > 0 && nums[i] == nums[i-1]) {
				continue;
			}
			for (int j = i+1; j < len; j++) {
				if (j > i+1 && nums[j] == nums[j-1]) {
					continue;
				}
				int sum = nums[i] + nums[j];
				if (sum > target && nums[j] > 0) {
					break;
				} else {
					int diff = target - sum;
					int m = j+1, n = len-1;
					while (m < n) {
						if (m > j+1 && nums[m] == nums[m-1]) {
							m++;
							continue;
						}
						int curSum = nums[m] + nums[n];
						if (curSum < diff) {
							m++;
						} else if (curSum > diff) {
							n--;
						} else {
							ret.add(Arrays.asList(nums[i], nums[j], nums[m], nums[n]));
							m++;
							n--;
						}
					}
				}
			}
		}

		return ret;
	}
```

<a id="problem_39"></a>
# 问题39 组合总和 ★★☆☆☆
https://leetcode-cn.com/problems/combination-sum/

这个题目和[问题40](#problem_39)很像，区别在于两点：
（1）问题39的入参数组无重复元素，问题40的入参数组可以包含重复元素；
（2）问题39中每个元素可以使用任意次，问题40中每个元素最多只能使用1次；
稍微思考一下，就能体会出为什么有这样的设定。因为问题39中每个元素可以使用任意次，所以即便是允许入参数组包含重复元素，那么也得先排序并过滤掉重复元素，题目里限定无重复元素等于是do you a favor. 问题40允许有重复元素但每个元素至多使用1次，其实就可以理解为问题39的变种，即可以理解为在问题39的基础上限定了每个元素最多使用几次。

需要注意的点是，如何保证结果集中无重复解呢？问题39是通过控制回溯过程来保证无重复解，而问题40是通过计数频次并控制每次回溯时同一个元素使用个数来保证无重复解。注意，对于问题40，如果仅仅是排序元素，然后按照类似问题39的思路去处理，即对每个当前遍历到的元素按不使用、使用分别继续下一回合遍历，这样是不行的，因为这样处理不能保证无重复解，比如，对于数组[1,2,2,2,2]，target=5，那么你会得到多个[1,2,2]的重复解。

另外，问题39和问题40这两个combination sum，和[问题1](#problem_1)、[问题167](#problem_167)、[问题15](#problem_15)、[问题16](#problem_16)、[问题18](#problem_18)这几个sum类问题是有一定相关性的。问题39和问题40可以看作问题1、问题167、问题15、问题16、问题18这几个sum类问题的进一步推广，即不限制相加之和等于目标值的子数组的元素个数。虽然从解法上有明显区别，但是这种区别也正是问题本身的扩展化和一般化带来的。

```java
  public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList<>();
    List<Integer> combination = new ArrayList<>();

    candidates = Arrays.stream(candidates).sorted().toArray();
    backtrack(result, combination, candidates, 0,  target);

    return result;
  }

  private void backtrack(List<List<Integer>> result,
                         List<Integer> combination,
                         int[] candidates,
                         int curIdx,
                         int target) {

    if (curIdx == candidates.length) {
      return;
    }

    if (target == 0) {
      result.add(new ArrayList<>(combination)); // pay attention here
      return;
    }

    int curVal = candidates[curIdx];
    if (curVal > target) {
      return;
    }

    // curIdx excluded
    backtrack(result, combination, candidates, curIdx+1, target);

    // curIdx included
    combination.add(curVal);
    backtrack(result, combination, candidates, curIdx, target-curVal);
    combination.remove(combination.size()-1);
  }
```

<a id="problem_40"></a>
# 问题40 组合总和 ★★☆☆☆
https://leetcode-cn.com/problems/combination-sum-ii/

题解参见[问题39](#problem_39)

```java
 public List<List<Integer>> combinationSum(int[] candidates, int target) {
    Arrays.sort(candidates);

    List<int[]> candidatesAndFreq = new ArrayList<>();
    Arrays.stream(candidates).forEach(e -> {
      if (candidatesAndFreq.size() > 0
          && candidatesAndFreq.get(candidatesAndFreq.size()-1)[0] == e) {
        candidatesAndFreq.get(candidatesAndFreq.size() - 1)[1] += 1;
      } else {
        int[] last = new int[2];
        last[0] = e;
        last[1] = 1;
        candidatesAndFreq.add(last);
      }
    });

    List<List<Integer>> result = new ArrayList<>();
    List<Integer> combination = new ArrayList<>();

    backtrack(result, combination, candidatesAndFreq, 0, target);

    return result;
  }

  private void backtrack(List<List<Integer>> result, List<Integer> combination,
                         List<int[]> candidates, int curIdx, int target) {

    if (target == 0) {
      result.add(new ArrayList<>(combination));
      return;
    }

    if (curIdx == candidates.size()) {
      return;
    }


    int curVal = candidates.get(curIdx)[0];
    int freq = candidates.get(curIdx)[1];

    if (curVal > target) {
      return;
    }

    for (int i = 0; i <= freq; i++) {
      if (target >= i * curVal) {
      	for (int j = 0; j < i; j++) {
          combination.add(curVal);
        }

        backtrack(result, combination, candidates, curIdx+1, target-i*curVal);

      	for (int j = 0; j < i; j++) {
          combination.remove(combination.size()-1);
        }
      }
    }
  }
```