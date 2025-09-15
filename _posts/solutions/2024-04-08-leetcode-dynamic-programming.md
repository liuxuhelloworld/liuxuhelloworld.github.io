<a id="problem_3"></a>
# 问题3 无重复字符的最长子串 ★☆☆☆☆
https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/

这个题目从动态规划的角度入手并不困难，定义dp[i]表示以s[i]结尾的具有最大非重复字符的子串的长度，那么dp[i]的最大值就是整个字符串中具有最大非重复字符的子串的长度。那么在dp[i]的基础上如何计算dp[i+1]呢？关键就是要确认s[i+1]这个字符在dp[i]对应的那个具有最大非重复字符的子串中的位置，如果s[i+1]不在其中，那么dp[i+1]=dp[i]+1，否则需要求差计算。

```java
    public int lengthOfLongestSubstringWithoutRepeatingCharacters(String s) {
        int len = s.length();
        if (len == 0) {
            return 0;
        }

        // dp[i] represents the length of longest substring that end with s[i] and has no repeating characters
        int[] dp = new int[len];
        dp[0] = 1;
        int max = 1;
        for (int i = 1; i < len; i++) {
            int last = lastIndexOf(s, i-dp[i-1],i-1, s.charAt(i));
            if (last == -1) {
                dp[i] = dp[i-1] + 1;
            } else {
                dp[i] = i-last;
            }
            if (dp[i] > max) {
                max = dp[i];
            }
        }

        return max;
    }

    private int lastIndexOf(String s, int start, int end, char c) {
        for (int i = end; i >= start; i--) {
            if (s.charAt(i) == c) {
                return i;
            }
        }

        return -1;
    }
```

<a id="problem_5"></a>
# 问题5 最长回文子串 ★☆☆☆☆
https://leetcode-cn.com/problems/longest-palindromic-substring/

这个题目算是比较典型的动态规划问题了吧。使用动态规划的关键是确定状态转移方程，即如何从小规模问题的解得到大一点规模问题的解，或者换句话说，如何把大一点规模的问题拆分为更小规模的问题。对于这个题目，s[i..j]是不是一个回文子串，可以拆分为首先判断s[i]是否等于s[j]，继而判断s[i+1..j-1]是不是一个回文子串，这样，我们就把一个大一点规模的问题拆分成了更小规模的问题，反过来，也就是我们可以从小规模问题的解，比较方便快捷地得到大一点规模问题的解。

另外，暴力解法见[问题5](2024-11-06-leetcode-string.md#problem_5)

```java
    public String longestPalindromeSubstring(String s) {
        if (s == null) {
            return null;
        }

        int len = s.length();

        String ret = "";

        // dp[i][j] represents whether s[i..j] is a palindromic substring
        boolean[][] dp = new boolean[len][len];

        for (int step = 0; step < len; step++) {
            for (int i = 0; i < len-step; i++) {
                if (step == 0) {
                    // s[i] is a palindromic substring of length one
                    dp[i][i] = true;
                } else if (step == 1) {
                    // s[i..i+1] is a palindromic substring of length two if s[i] == s[i+1]
                    dp[i][i+1] = s.charAt(i) == s.charAt(i+1);
                } else {
                    // s[i..i+step] is a palindromic substring of length step
                    // if s[i] == s[i+step] and s[i+1..i+step-1] is also a palindromic substring
                    dp[i][i+step] = dp[i+1][i+step-1] && s.charAt(i) == s.charAt(i+step);
                }

                if (dp[i][i+step] && step+1 > ret.length()) {
                    ret = s.substring(i, i+step+1);
                }
            }
        }

        return ret;
    }
```

# 问题45 跳跃游戏 ★☆☆☆☆
https://leetcode-cn.com/problems/jump-game-ii/

这个题目如果从递归的思路来看，还是有比较清晰的解题思路的，可以倒序遍历数组，对于能一步跳到结尾元素的X，问题变为从开头元素跳到X的最小跳数，而这恰好是原问题的更小规模的子问题，由此递归直到开头元素。

一个最优解问题，能通过递归解决，递归过程中有可能重复处理子问题，这些都提示应当使用动态规划。动态规划的关键是定义合适的状态转移方程，这个题目里我们可以定义dp[i]表示从i元素到结尾元素的最小跳数，与上述递归思路类似，同样是倒叙遍历数组，dp[i]的值可以从dp[i+1]到dp[len-1]的值计算而得。

从提交记录看，这种解法的效率很不错。

```java
	public int jump(int[] nums) {
		int len = nums.length;

		if (len == 0 || len == 1) {
			return 0;
		}

		int[] jumps = new int[len]; // jumps[i] means the minimum jumps from nums[i] to the last element
		for (int i = 0; i < len-1; i++) {
			jumps[i] = -1;
		}
		jumps[len-1] = 0;

		for (int i = len-2; i >= 0; i--) {
			jumps[i] = minJumps(jumps, i, nums[i]);
		}

		return jumps[0];
	}

	private int minJumps(int[] jumps, int i, int val) {
		if (val == 0) {
			return -1;
		}

		int min = Integer.MAX_VALUE;
		for (int j = i + 1; j <= i + val && j < jumps.length; j++) {
			if (jumps[j] == -1) {
				continue;
			}
			min = Math.min(min, 1 + jumps[j]);
		}

		if (min == Integer.MAX_VALUE) {
			return -1;
		}

		return min;
	}
```

# 问题70 爬楼梯 ★☆☆☆☆
https://leetcode-cn.com/problems/climbing-stairs/

从动态规划的角度去考虑，还是很容易想到怎么解决这个问题的。从第1层到第i层，每次只能走1层或2层，所以无论怎么走，最终到第i层的可能走法就是到第i-2层的可能走法加上到第i-1层的可能走法之和。也就是说，就是Fibonacci数列。

```java
	public int climbStairs(int n) {
		if (n == 1) {
			return 1;
		}

		if (n == 2) {
			return 2;
		}

		int[] dp = new int[n + 1];
		dp[1] = 1;
		dp[2] = 2;

		for (int i = 3; i <= n; i++) {
			dp[i] = dp[i - 1] + dp[i - 2];
		}

		return dp[n];
	}
```

# 问题198 打家劫舍 ★☆☆☆☆
https://leetcode-cn.com/problems/house-robber/

这个题目从动态规划的角度考虑，并不难想到如何解决。我们定义一个dp数组，dp[i]表示num前i个元素对应的最大抢劫金钱数。那么如何从较小规模的dp元素值得到dp[i]的值呢？如果dp[i]对应的结果包含num第i个元素，那么最多抢劫到num[i-1]+dp[i-2]的钱；如果dp[i]对应的结果不包含num第i个元素，那么最多抢劫到dp[i-1]的钱。所以，dp[i]的值就是这两者中的较大者。

```java
	public int rob(int[] nums) {
		int len = nums.length;

		int[] dp = new int[len+1];
		dp[0] = 0;
		dp[1] = nums[0];

		for (int i = 2; i <= len; i++) {
			int robWithI = nums[i-1] + dp[i-2];
			int robWithoutI = dp[i-1];
			dp[i] = Math.max(robWithI, robWithoutI);
		}

		return dp[len];
	}
```

# 问题91 解码方法 ★☆☆☆☆
https://leetcode-cn.com/problems/decode-ways/

对于这个题目，并不难想到用动态规划来解决，dp[i]表示长度为i的子串的解码方式数量，dp[1]根据s[0..0]是不是合法的编码值而初始化为1或0 ，dp[i+1]的值由dp[i], dp[i-1], s[i-1..i-1], s[i-2..i-1]四个值决定，代码如下：

```java
	public int numDecodings(String s) {
		assert s != null && s.length() > 0;

		int len = s.length();

		int[] dp = new int[len + 1];

		if (isValid(s.substring(0, 1))) {
			dp[1] = 1;
		} else {
			return 0;
		}

		for (int i = 2; i <= len; i++) {
			String sub = s.substring(i-1, i);
			if (isValid(sub)) {
				dp[i] += dp[i-1];
			}

			sub = s.substring(i-2, i);
			if (isValid(sub)) {
				dp[i] += dp[i-2];
			}

			if (dp[i] == 0) {
				return 0;
			}
		}

		return dp[len];
	}

	private boolean isValid(String s) {
		int len = s.length();

		if (len == 0 || len > 2) {
			return false;
		}

		int val = Integer.parseInt(s);
		if (len == 1 && val != 0) {
			return true;
		}
		if (len == 2 && val >= 10 && val <= 26) {
			return true;
		}

		return false;
	}
```

# 问题337 打家劫舍 ★★☆☆☆
https://leetcode.cn/problems/house-robber-iii/

这个题目有点意思，一开始我没搞清楚题目的意思，认为可行的解法是：给定一个节点，获取左子树的最大rob数，获取右子树的最大rob数，如果左子树和右子树最大rob数之和大于当前节点的val，那么返回它们的和作为当前节点的子树的最大rob数，否则，返回val作为当前节点的最大rob数。递归很好写嘛，但是，以题目中的示例去验证就会发现问题，这种解法当然满足题目中节点不相邻的限制，但不是最优解。

那么应该如何处理呢？给定一个节点，我们有两种选择，一种是选定这个节点，一种是不选定这个节点，我们需要的其实是这样的信息：在选定这个节点时，这颗子树的最大rob数是多少？在不选定这个节点时，这颗子树的最大rob数是多少？即\[selected, unselected\]。如果有了这样的信息，那么给定一个节点，我们可以先分别计算它的左子树和右子树，得到\[leftSelected, leftUnselected\]和\[rightSelected, rightUnselected\]，那么对于当前节点，selected = val + leftUnselected + rightUnselected，unselected = max(leftSelected, leftUnselected)+max(rightSelected, rightUnselected)，以此递归可得根节点的\[selected, unselected\]，它们中的较大值即为最优解。

```java
	public int rob(TreeNode root) {
		int[] rob = dfs(root);
		return Math.max(rob[0], rob[1]);
	}

	private int[] dfs(TreeNode node) {
		if (node == null) {
			return new int[] {0, 0};
		}

		int[] left = dfs(node.left);
		int[] right = dfs(node.right);
		int val = node.val;

		int leftSelected = left[0];
		int leftUnselected = left[1];
		int rightSelected = right[0];
		int rightUnselected = right[1];
		int nodeSelected = val + leftUnselected + rightUnselected;
		int nodeUnselected = Math.max(leftSelected, leftUnselected) + Math.max(rightSelected, rightUnselected);

		return new int[] {nodeSelected, nodeUnselected};
	}
```

# 问题120 三角形最小路径和 ★★☆☆☆
https://leetcode-cn.com/problems/triangle/

从动态规划的角度去考虑，并不难想到如何解决这个问题。维护一个二维数组，dp\[i\]\[j]表示从top节点到第i行第j列的节点的最小路径sum，而dp\[i\]\[j\] = min(dp\[i-1\]\[j-1\], dp\[i-1\]\[j\]) + triangle\[i\]\[j\]，这样从二维数组的最后一行中找到最小值就是从top节点到最后一行的最小路径sum. 题目中进一步提到是不是可以只使用O(n)的额外空间，其实想一下，dp\[i\]这一行的计算只依赖dp\[i-1\]这一行，所以是可以只维护一个一维数组，这个数组存储的是从top节点到当前遍历到的那一行对应的各个元素的最小路径sum. 进一步的，遍历下一行时如何更新这个一维数组呢？dp\[i\]\[j\] = min(dp\[i-1\]\[j-1\], dp\[i-1\]\[j\]) + triangle\[i\]\[j\]，所以只能从右到左进行更新，从左到右更新会覆盖掉还需要进一步使用的原数据。注意dp数组的初始化逻辑和triangle第一行的处理。

```java
	public int minimumTotal(List<List<Integer>> triangle) {
		int rows = triangle.size();

		int[] dp = new int[rows + 1];
		for (int i = 0; i <= rows; i++) {
			dp[i] = Integer.MAX_VALUE;
		}

		List<Integer> first = triangle.get(0);
		dp[1] = first.get(0);

		for (int i = 1; i < rows; i++) {
			List<Integer> cur = triangle.get(i);

			for (int j = i + 1; j >= 1; j--) {
				dp[j] = Math.min(dp[j], dp[j-1]) + cur.get(j-1);
			}
		}

		int min = dp[1];
		for (int i = 1; i <= rows; i++) {
			if (dp[i] < min) {
				min = dp[i];
			}
		}

		return min;
	}
```

# 问题62 不同路径 ★★☆☆☆
https://leetcode.cn/problems/unique-paths/

如果能想到以动态规划为思路来解决这个题目，那么这个题目并不难。状态转移方程也很符合直觉，定义dp\[i\]\[j]表示从[0, 0]到[i, j]的路径数，dp\[i\]\[j\]的计算只依赖于dp\[i-1\]\[j\]和dp\[i\]\[j-1\]。

值得注意的是，这个题目容易给人一种适合于使用回溯法的错觉，使用回溯法也能计算出正确结果，但是代码实现起来复杂，时间效率也很差。

```java
	public int uniquePaths(int m, int n) {
		int[][] dp = new int[m][n]; // dp[i][j] represents the unique paths from [0,0] to [i,j]
		for (int i = 0; i < m; i++) {
			dp[i][0] = 1;
		}
		for (int j = 0; j < n; j++) {
			dp[0][j] = 1;
		}

		for (int i = 1, j = 1; i < m && j < n; i++, j++) {
			for (int k = j; k < n; k++) {
				dp[i][k] = dp[i-1][k] + dp[i][k-1];
			}
			for (int k = i; k < m; k++) {
				dp[k][j] = dp[k-1][j] + dp[k][j-1];
			}
		}

		return dp[m-1][n-1];
	}
```

# 问题1105 填充书架 ★★☆☆☆
https://leetcode.cn/problems/filling-bookcase-shelves/

这个题目，坦白讲，光理解题目意图就花了一定的时间，但是总结起来也不复杂。给定一组书，每本书有宽度和高度两个属性，还有一个固定宽度的书架，现在要把这些书按给定的顺序摆放到书架上，摆不开了就换层，求能实现摆放完所有书的最小总层高是多少。这里面有一个关键点是要按给定的顺序摆放到书架上。从直觉上，要想让总层高最小，那么就应该尽量把高度大的书摆放到同一层，因为每一层的高度是由这一层中最高的书决定的，如果高度大的书摆放到了不同层，那么显然有悖于目标。但是由于限制要按给定的顺序摆放，不能调整书的顺序而把高度大的书尽量放到同一层，所以这就涉及到了在哪里去开始下一层的问题。

那么如何得到最小总层高呢？这种求最小值的问题，直觉上容易想到动态规划。动态规划的关键是由小到大的解决问题，入手点就是推理问题由小到大的过程中如何利用小规模问题的解得到大规模问题的解。

这个题目我们可以这样推理：处理第一本书，那么显然最小总层高就是这本书的高度，我们记为dp\[0\]。处理第二本书，如果不换层，那么总层高是第一本书和第二本书高度的较大值，如果换层，那么总层高是第一本书和第二本书高度之和，显然不换层是更好的选择，但是是不是可以不换层要看两本书的宽度是否满足书架宽度的限制，如果一层放不下，只能换层，如果一层能放下，显然就不换层，由此我们可以得到处理完第二本书后的最小总层高，我们记为dp\[1\]。处理第三本书，直接换层，加上dp\[1\]可得到一个层高；尝试将第二本书与第三本书摆放到同一层，加上dp\[0\]可得到一个层高；尝试将第二本书、第一本书与第三本书均摆放到同一层，可得到一个层高；这个过程均需验证是否满足书架宽度限制，由此我们可以得到处理完第三本书后的最小总层高，我们记为dp\[2\]。以此类推，处理到第i本书时，为了得到dp\[i-1\]，我们需要从第i本书倒序遍历，直到因书架宽度限制而不能与第i本书摆放到同一层的那一本，在这个倒序遍历过程中，我们需要计算每种可能下的层高，最终得到最小总层高dp\[i-1\]。

```java
	public int minHeightShelves(int[][] books, int shelfWidth) {
		int length = books.length;

		int[] thick = new int[length];
		int[] height = new int[length];

		for (int i = 0; i < length; i++) {
			thick[i] = books[i][0];
			height[i] = books[i][1];
		}

		// dp[i] represents min height to fill books[0..i]
		int[] dp = new int[length];
		dp[0] = height[0];

		for (int i = 1; i < length; i++) {
			int thickSum = 0;
			int maxBookHeight = height[i];
			int minShelfHeight = Integer.MAX_VALUE;
			for (int k = i; k >= 0; k--) {
				thickSum += thick[k];
				if (height[k] > maxBookHeight) {
					maxBookHeight = height[k];
				}

				if (thickSum > shelfWidth) {
					break;
				}

				if (k == 0) {
					if (maxBookHeight < minShelfHeight) {
						minShelfHeight = maxBookHeight;
					}
				} else {
					if (dp[k - 1] + maxBookHeight < minShelfHeight) {
						minShelfHeight = dp[k - 1] + maxBookHeight;
					}
				}
			}

			dp[i] = minShelfHeight;
		}

		return dp[length-1];
	}
```

# 问题139 单词拆分 ★★★☆☆
https://leetcode-cn.com/problems/word-break/

这个题目并不难想到使用动态规划，关键是状态转移方程。我们定义dp[i]表示字符串s的前i个字符对应的子串是否满足wordBreak，那么怎么得到dp[i+1]呢？如果dp[i]是true并且wordDict包含s[i, i+1]，那么dp[i+1]为true，或者如果dp[i-1]是true并且wordDict包含s[i-1, i+1]，那么dp[i+1]为true，或者如果dp[i-2]是true并且wordDict包含s[i-2, i+1]，那么dp[i+1]为true，......，或者如果dp[0]是true并且wordDict包含s[0, i+1]，那么dp[i+1]为true.

```java
	public boolean wordBreak(String s, List<String> wordDict) {
		int length = s.length();

		boolean[] dp = new boolean[length + 1]; // dp[i] represents whether s[0..i-1] fits wordBreak
		dp[0] = true;

		for (int i = 1; i <= length; i++) {
			for (int j = i - 1; j >= 0; j--) {
				if (dp[j] && wordDict.contains(s.substring(j, i))) {
					dp[i] = true;
					break;
				}
			}
		}

		return dp[length];
	}
```

# 问题97 交错字符串 ★★★☆☆
https://leetcode-cn.com/problems/interleaving-string/

我觉得这个题目有点难度，光是题目要求就需要理解消化，将两个字符串分别划分为两个子串序列，子串个数相等或者相差为1，然后把两个子串序列交错的拼接成一个字符串，现在给定三个字符串，问是否存在某种划分方式，使得划分以后得到的两个子串序列交错以后可以组成第三个字符串。

这个题目我一开始的思路是递归，尝试编码以后发现超时。其实如果有更好的解题直觉，直接从动态规划入手，状态转移方程并不难想到。我们定义dp\[m\]\[n\]，其中dp\[i\]\[j\]表示s1的前i个字符和s2的前j个字符是否能划分并交错组成s3的前i+j个字符。那么dp\[i\]\[j\]的计算和dp\[i-1\]\[j\]、dp\[i\]\[j-1\]、s1[i-1]、s2[j-1]、s3[i+j-1]有关。

```java
	public boolean isInterleave(String s1, String s2, String s3) {
		assert s1 != null && s2 != null && s3 != null;

		int len1 = s1.length();
		int len2 = s2.length();
		int len3 = s3.length();

		if (len1 + len2 != len3) {
			return false;
		}

		if (len1 == 0 && len2 == 0 && len3 == 0) {
			return true;
		}

		boolean[][] dp = new boolean[len1 + 1][len2 + 1];

		int matches1 = matches(s1, s3);
		for (int len = 1; len <= matches1; len++) {
			dp[len][0] = true;
		}

		int matches2 = matches(s2, s3);
		for (int len = 1; len <= matches2; len++) {
			dp[0][len] = true;
		}

		for (int sum = 2; sum <= len1 + len2; sum++) {
			for (int i = 1; i <= sum; i++) {
				int j = sum - i;

				if (i <= 0 || i > len1 || j <= 0 || j > len2) {
					continue;
				}

				if (s1.charAt(i - 1) == s3.charAt(i + j - 1)) {
					dp[i][j] = dp[i][j] || dp[i - 1][j];
				}
				if (s2.charAt(j - 1) == s3.charAt(i + j - 1)) {
					dp[i][j] = dp[i][j] || dp[i][j - 1];
				}
			}
		}

		return dp[len1][len2];
	}
	
		private int matches(String s, String t) {
		int cnt = 0;
		for (int i = 0, j = 0; i < s.length() && j < t.length(); i++, j++) {
			if (s.charAt(i) == t.charAt(j)) {
				cnt++;
			} else {
				break;
			}
		}

		return cnt;
	}
```