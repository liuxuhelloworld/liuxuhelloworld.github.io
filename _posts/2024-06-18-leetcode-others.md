# 问题2319 判断矩阵是否是一个X矩阵 ☆☆☆☆☆
https://leetcode.cn/problems/check-if-matrix-is-x-matrix/

这个题目比较简单了，稍微归纳一下，可以发现，对角线元素就是那些i == j或者i+j == n-1的元素，由此双循环遍历并判断即可。

```java
	public boolean checkXMatrix(int[][] grid) {
		int n = grid.length;

		for (int i = 0; i < n; i++) {
			for (int j = 0; j < n; j++) {
				if (i == j || i+j == n-1) {
					if (grid[i][j] == 0) {
						return false;
					}
				} else {
					if (grid[i][j] != 0) {
						return false;
					}
				}
			}
		}

		return true;
	}
```

# 问题2347 最好的扑克手牌 ☆☆☆☆☆
https://leetcode.cn/problems/best-poker-hand/

这个题目确实比较简单了，根据题目要求，Flush只需要判断花色，Three of a Kind、Pair、High Card这些只需要判断出现频次最高的点数。

```java
	public String bestHand(int[] ranks, char[] suits) {
		boolean isFlush = isFlush(suits);
		if (isFlush) {
			return "Flush";
		}

		int[] count = new int[14]; // 0 index is not used
		for (int i = 0; i < ranks.length; i++) {
			count[ranks[i]]++;
		}

		int max = 0;
		for (int i = 0; i < count.length; i++) {
			if (count[i] > max) {
				max = count[i];
			}
		}

		if (max >= 3) {
			return "Three of a Kind";
		} else if (max == 2) {
			return "Pair";
		} else {
			return "High Card";
		}
	}

	private boolean isFlush(char[] suits) {
		for (int i = 1; i < suits.length; i++) {
			if (suits[i] != suits[i-1]) {
				return false;
			}
		}

		return true;
	}
```

# 问题344 反转字符串 ☆☆☆☆☆
https://leetcode-cn.com/problems/reverse-string/

无需赘述。

```java
	public void reverseString(char[] s) {
		for (int i = 0, j = s.length - 1; i < j; i++, j--) {
			char tmp = s[i];
			s[i] = s[j];
			s[j] = tmp;
		}
	}
```

# 问题557 反转字符串中的单词 ★☆☆☆☆
https://leetcode-cn.com/problems/reverse-words-in-a-string-iii/

这个题目从思路上讲，没有什么难度，最直接的处理可以把字符串转换为字符数组，然后遍历该数组，如果不是空格，将字符加到一个临时字符串中；如果是空格，则反转临时字符串并添加到结果字符串中，然后把空格也加入结果字符串。

这里使用了库函数，代码简洁很多。

```java
	public String reverseWords(String s) {
		String[] words = s.split(" ");
		String[] reversed = Arrays.stream(words)
			.map(e -> reverse(e))
			.collect(Collectors.toList())
			.toArray(new String[0]);

		return String.join(" ", reversed);
	}

	private String reverse(String s) {
		StringBuilder rev = new StringBuilder();
		for (int i = s.length()-1; i >= 0; i--) {
			rev.append(s.charAt(i));
		}
		return rev.toString();
	}
```

# 问题844 比较含退格的字符串 ★☆☆☆☆
https://leetcode-cn.com/problems/backspace-string-compare/

这个题目最直接的思路就是按照backspace规则，分别把两个字符串做backspace处理，然后比较得到的结果。代码并不复杂，提交以后也通过了。这种处理的时间复杂度和空间复杂度都是O(N).

那么题目中附加的能不能以O(1)的空间复杂度解答这个题目呢？参考了官方题解才恍然大悟，backspace只可能影响左边的字符，所以倒序处理字符串，就能增量的挨个比较下一个能保留下来的字符。换句话说，我们倒序遍历两个字符串，每次遍历的目标是获得下一个能保留下来的字符的下标，然后比较两个字符串下一个能保留下来的字符是否相同，相同则继续下一轮遍历，不同或有一方为空了，那么说明两个字符串backspace后的字符串不同。

```java
	public boolean backspaceCompare(String s, String t) {
		char[] schars = s.toCharArray();
		char[] tchars = t.toCharArray();

		int i = schars.length, j = tchars.length;
		while (true) {
			i = idxOfNextRightMostChar(schars, i);
			j = idxOfNextRightMostChar(tchars, j);
			if (i == -1 && j == -1) {
				return true;
			} else if (i != -1 && j != -1) {
				char ch1 = schars[i];
				char ch2 = tchars[j];
				if (ch1 != ch2) {
					return false;
				}
			} else {
				return false;
			}
		}
	}

	private int idxOfNextRightMostChar(char[] chars, int idx) {
		int skip = 0;

		for (int i = --idx; i >= 0; i--) {
			char ch = chars[i];
			if (ch == '#') {
				skip++;
			} else {
				if (skip > 0) {
					skip--;
				} else {
					return i;
				}
			}
		}

		return -1;
	}
```

# 问题2325 解密消息 ★☆☆☆☆
https://leetcode.cn/problems/decode-the-message/

这个题目比较简单了，关键是先维护出一个解密table，不赘述。

```java
	public String decodeMessage(String key, String message) {
		char[] table = new char[26];
		for (int i = 0; i < 26; i++) {
			table[i] = 0;
		}

		char[] keyCharArr = key.toCharArray();
		char ch = 'a';
		for (int i = 0; i < keyCharArr.length; i++) {
			char cur = keyCharArr[i];
			if (cur != ' ' && table[cur - 'a'] == 0) {
				table[cur - 'a'] = ch;
				ch = (char)(ch + 1);
			}
		}

		char[] messageCharArr = message.toCharArray();
		char[] decodedCharArr = new char[messageCharArr.length];
		for (int i = 0; i < messageCharArr.length; i++) {
			char cur = messageCharArr[i];
			if (cur != ' ') {
				decodedCharArr[i] = table[cur - 'a'];
			} else {
				decodedCharArr[i] = ' ';
			}
		}

		return String.valueOf(decodedCharArr);
	}
```

# 问题189 轮转数组 ★☆☆☆☆
https://leetcode-cn.com/problems/rotate-array/

这个题目比较直接的想法就是把右侧的k个元素暂存下来，右移处理左侧len-k个元素，这样左侧空出k个位置，再把暂存的右侧k个元素填补上即可。这种处理方法的空间复杂度是O(N).

如果期望以O(1)的空间复杂度来解决呢？简单一点的办法是每次右移1个元素，操作k次即可。

有没有更好的办法呢？额，对于连续reverse这种解决方法，背下来好了......

```java
	public void rotate(int[] nums, int k) {
		int len = nums.length;
		if (len == 0) {
			return;
		}

		k = k % len;
		if (k == 0) {
			return;
		}

		// reverse left len-k elements
		reverse(nums, 0, len - k - 1);

		// reverse right k elements
		reverse(nums, len - k, len - 1);

		// reverse all elements
		reverse(nums, 0, len - 1);
	}

	private void reverse(int[] nums, int left, int right) {
		for (int i = left, j = right; i < j; i++, j--) {
			int tmp = nums[i];
			nums[i] = nums[j];
			nums[j] = tmp;
		}
	}
```

# 问题2352 相等行列对 ☆☆☆☆☆
https://leetcode.cn/problems/equal-row-and-column-pairs/

题目本身并不难理解，给定一个n*n的二维数组，按行列去找相等的pair，换句话说，用每一行和每一列来组合并判断是否相等。按照这样的思路，直接双重循环就可以了。判断某一行和某一列是否相等时，注意early return，碰到第一个不相等的元素即可返回，想来效率不会太差。但是，考虑到题目难度设置是中等，是不是有更高效的解法呢？思考了下，实在没有思路，先写个暴力解法的再说。代码提交以后没有问题，而且效率也还可以。看了下官方题解，其中对行和列做某种形式转换再基于哈希表做查询计数的思路有一定启发性，但我还是想说一句，“何必呢”。

```java
	public int equalPairs(int[][] grid) {
		int length = grid.length;

		int cnt = 0;
		for (int i = 0; i < length; i++) {
			for (int j = 0; j < length; j++) {
				if (isEqual(grid, i, j)) {
					cnt++;
				}
			}
		}

		return cnt;
	}

	private boolean isEqual(int[][] grid, int rowIdx, int columnIdx) {
		int[] row = grid[rowIdx];
		for (int i = 0; i < row.length; i++) {
			if (row[i] != grid[i][columnIdx]) {
				return false;
			}
		}

		return true;
	}
```

# 问题2383 赢得比赛需要的最少训练时长 ★☆☆☆☆
https://leetcode.cn/problems/minimum-hours-of-training-to-win-a-competition/

这个题目比较简单了，虽然题目描述文字一大堆，但是逻辑并不复杂。按顺序遍历数组，遍历过程中根据当前的energy和experience，以及opponent的energy和experience，可以计算出是否缺少足够的energy或experience、缺少多少，计数缺少的总数，更新进入下一轮遍历前的energy和experience，然后继续遍历。

```java
	public int minNumberOfHours(int initialEnergy, int initialExperience, int[] energy, int[] experience) {
		int lackedEnergySum = 0, lackedExperienceSum = 0;
		int curEnergy = initialEnergy, curExperience = initialExperience;
		for (int i = 0; i < energy.length; i++) {
			int opponentEnergy = energy[i];
			int opponentExperience = experience[i];

			if (curEnergy <= opponentEnergy) {
				int lackedEnergy = (opponentEnergy - curEnergy) + 1;
				curEnergy += lackedEnergy;
				lackedEnergySum += lackedEnergy;
			}

			curEnergy = curEnergy - opponentEnergy;

			if (curExperience <= opponentExperience) {
				int lackedExperience = (opponentExperience - curExperience) + 1;
				curExperience = curExperience + lackedExperience;
				lackedExperienceSum += lackedExperience;
			}

			curExperience = curExperience + opponentExperience;
		}

		return lackedEnergySum + lackedExperienceSum;
	}
```

# 问题1222 可以攻击国王的皇后 ★☆☆☆☆
https://leetcode.cn/problems/queens-that-can-attack-the-king/

结合示例，题目并不难理解，一个国际象棋棋盘，有一个位置是king，其他一些位置是queen，沿着横向、纵向、对角三条线，离king最近的queen可以直接攻击king，注意必须是最近的，不能隔着queen去攻击king，现在需要找出这些能直接攻击king的queen。

解题思路呢？我觉得很直接嘛，从king的位置开始，沿着向右、向右上、向上、向左上、向左、向左下、向下、向右下8个方向，挨个方向遍历就好了，碰到第一个queen就是能直接攻击king的queue，那么这个方向的遍历就可以退出了。但是题目的入参不是一个二维数组，而是queen和king的位置，没关系，我们自己构造一个二维数组就好了。

```java
	public List<List<Integer>> queensAttacktheKing(int[][] queens, int[] king) {
		int[][] board = new int[8][8];

		for (int[] queen : queens) {
			int queenRow = queen[0];
			int queenCol = queen[1];
			board[queenRow][queenCol] = 1;
		}

		int kingRow = king[0];
		int kingCol = king[1];
		board[kingRow][kingCol] = 2;

		List<List<Integer>> attacks = new ArrayList<>();

		// go right
		for (int j = kingCol + 1; j < 8; j++) {
			if (board[kingRow][j] == 1) {
				attacks.add(Arrays.asList(kingRow, j));
				break;
			}
		}

		// go up-right
		for (int i = kingRow - 1, j = kingCol + 1; i >= 0 && j < 8; i--, j++) {
			if (board[i][j] == 1) {
				attacks.add(Arrays.asList(i, j));
				break;
			}
		}

		// go up
		for (int i = kingRow - 1; i >= 0; i--) {
			if (board[i][kingCol] == 1) {
				attacks.add(Arrays.asList(i, kingCol));
				break;
			}
		}

		// go up-left
		for (int i = kingRow - 1, j = kingCol - 1; i >= 0 && j >= 0; i--, j--) {
			if (board[i][j] == 1) {
				attacks.add(Arrays.asList(i, j));
				break;
			}
		}

		// go left
		for (int j = kingCol - 1; j >= 0; j--) {
			if (board[kingRow][j] == 1) {
				attacks.add(Arrays.asList(kingRow, j));
				break;
			}
		}

		// go down-left
		for (int i = kingRow + 1, j = kingCol - 1; i < 8 && j >= 0; i++, j--) {
			if (board[i][j] == 1) {
				attacks.add(Arrays.asList(i, j));
				break;
			}
		}

		// go down
		for (int i = kingRow + 1; i < 8; i++) {
			if (board[i][kingCol] == 1) {
				attacks.add(Arrays.asList(i, kingCol));
				break;
			}
		}

		// go down-right
		for (int i = kingRow + 1, j = kingCol + 1; i < 8 && j < 8; i++, j++) {
			if (board[i][j] == 1) {
				attacks.add(Arrays.asList(i, j));
				break;
			}
		}

		return attacks;
	}
```

# 问题1604 警告一小时内使用相同员工卡大于等于三次的人 ★☆☆☆☆
https://leetcode.cn/problems/alert-using-same-key-card-three-or-more-times-in-a-one-hour-period/

这个题目从思路来说我觉得并不难，属于比较侧重代码书写的题目。按用户归集打卡时间后遍历，遍历过程中先排序，然后判断是否存在一个小时内三次或以上打开记录。因为目标是三次或以上，直接隔一个元素判断就可以。

```java
	public List<String> alertNames(String[] keyName, String[] keyTime) {
		Map<String, List<String>> nameTimeMap = new HashMap<>();
		for (int i = 0; i < keyName.length; i++) {
			String name = keyName[i];
			String time = keyTime[i];
			if (nameTimeMap.containsKey(name)) {
				nameTimeMap.get(name).add(time);
			} else {
				List<String> list = new ArrayList<>();
				list.add(time);
				nameTimeMap.put(name, list);
			}
		}

		List<String> alertNameList = new ArrayList<>();

		for (Map.Entry<String, List<String>> entry : nameTimeMap.entrySet()) {
			String name = entry.getKey();
			List<String> timeList = entry.getValue();

			Collections.sort(timeList);

			if (existsThreeOrMoreTimesInOneHourPeriod(timeList)) {
				alertNameList.add(name);
			}
		}

		Collections.sort(alertNameList);

		return alertNameList;
	}

	private boolean existsThreeOrMoreTimesInOneHourPeriod(List<String> timeList) {
		String[] timeArray = timeList.toArray(new String[] {});
		for (int i = 2; i < timeArray.length; i++) {
			String cur = timeArray[i];
			String twoPositionsBefore = timeArray[i - 2];
			if (isOneHourPeriod(twoPositionsBefore, cur)) {
				return true;
			}
		}

		return false;
	}

	private boolean isOneHourPeriod(String time1, String time2) {
		int hour1 = Integer.parseInt(time1.substring(0, 2));
		int minute1 = Integer.parseInt(time1.substring(3, 5));
		int hour2 = Integer.parseInt(time2.substring(0, 2));
		int minute2 = Integer.parseInt(time2.substring(3, 5));

		int hourDiff = Math.abs(hour1 - hour2);
		if (hourDiff == 0) {
			return true;
		} else if (hourDiff == 1) {
			if (minute1 >= minute2) {
				return true;
			} else {
				return false;
			}
		} else {
			return false;
		}
	}
```

# 问题2409 统计共同度过的日子数 ☆☆☆☆☆
https://leetcode.cn/problems/count-days-spent-together/

这个题目从解题思路来说，确实没有什么难度，稍作分析，可以确定，取Alice和Bob较大的到达日期作为起始日期，取Alice和Bob较小的离开日期作为结束日期，然后计算起始日期到结束日期包含的天数即可。因为日期的格式是固定的，直接取对应子串即可解析出month和day，计算包含的天数时注意可能跨月。

```java
	private static final int[] DAYS_PER_MONTH = new int[] {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};

	public int countDaysTogether(String arriveAlice, String leaveAlice, String arriveBob, String leaveBob) {
		int arriveAliceMonth = month(arriveAlice);
		int arriveAliceDay = day(arriveAlice);
		int leaveAliceMonth = month(leaveAlice);
		int leaveAliceDay = day(leaveAlice);
		int arriveBobMonth = month(arriveBob);
		int arriveBobDay = day(arriveBob);
		int leaveBobMonth = month(leaveBob);
		int leaveBobDay = day(leaveBob);

		int biggerArriveMonth, biggerArriveDay;
		int cmp = compare(arriveAliceMonth, arriveAliceDay, arriveBobMonth, arriveBobDay);
		if (cmp < 0) {
			biggerArriveMonth = arriveBobMonth;
			biggerArriveDay = arriveBobDay;
		} else {
			biggerArriveMonth = arriveAliceMonth;
			biggerArriveDay = arriveAliceDay;
		}

		int smallerLeaveMonth, smallerLeaveDay;
		cmp = compare(leaveAliceMonth, leaveAliceDay, leaveBobMonth, leaveBobDay);
		if (cmp < 0) {
			smallerLeaveMonth = leaveAliceMonth;
			smallerLeaveDay = leaveAliceDay;
		} else {
			smallerLeaveMonth = leaveBobMonth;
			smallerLeaveDay = leaveBobDay;
		}

		return days(biggerArriveMonth, biggerArriveDay, smallerLeaveMonth, smallerLeaveDay);
	}

	private int compare(int month1, int day1, int month2, int day2) {
		int cmp = month1 - month2;
		if (cmp != 0) {
			return cmp;
		}

		return day1 - day2;
	}

	private int days(int startMonth, int startDay, int endMonth, int endDay) {
		int cmp = compare(startMonth, startDay, endMonth, endDay);
		if (cmp > 0) {
			return 0;
		} else if (cmp == 0) {
			return 1;
		} else {
			if (startMonth == endMonth) {
				return endDay - startDay + 1;
			} else {
				int days = 0;
				days += DAYS_PER_MONTH[startMonth] - startDay + 1;
				for (int i = startMonth + 1; i < endMonth; i++) {
					days += DAYS_PER_MONTH[i];
				}
				days += endDay;

				return days;
			}
		}
	}

	private int month(String date) {
		return Integer.parseInt(date.substring(0, 2));
	}

	private int day(String date) {
		return Integer.parseInt(date.substring(3, 5));
	}
```

# 问题2437 有效时间的数目 ☆☆☆☆☆
https://leetcode.cn/problems/number-of-valid-clock-times/

题目的解答思路不难，但我觉得这个题目还是能考察下代码能力的，尤其是能不能把代码写得简单清晰易理解。

```java
	public int countTime(String time) {
		char[] chars = time.toCharArray();
		boolean hourHighQuestionMark = chars[0] == '?';
		boolean hourLowQuestionMark = chars[1] == '?';
		boolean minuteHighQuestionMark = chars[3] == '?';
		boolean minuteLowQuestionMark = chars[4] == '?';

		int hours = 1;
		if (hourHighQuestionMark && hourLowQuestionMark) {
			hours = 24;
		} else {
			if (hourHighQuestionMark) {
				if (chars[1] > '3') {
					hours = 2;
				} else {
					hours = 3;
				}
			} else if (hourLowQuestionMark) {
				if (chars[0] < '2') {
					hours = 10;
				} else {
					hours = 4;
				}
			}
		}

		int minutes = 1;
		if (minuteHighQuestionMark) {
			minutes *= 6;
		}
		if (minuteLowQuestionMark) {
			minutes *= 10;
		}

		return hours * minutes;
	}
```

# 问题2596 检查骑士巡视方案 ★☆☆☆☆
https://leetcode.cn/problems/check-knight-tour-configuration/

这个题目乍一看描述有点不好理解，但结合示例就比较清楚了，简单说就是给定一个二维数组，每个元素的值表示第X步，注意走一步的规则是行列坐标加减2和加减1的组合，即下一步最多可能有8个备选位置，为什么是最多呢？因为可能有一些已经超出二维数组的合法下标范围。问题来了，对于给定的走法，是不是一个有效的能遍历所有位置一遍的走法呢？

我的解题思路比较直接，既然是判断给定的走法是不是有效的走法，那就模拟着走一遍嘛。起始位置是\[0,0\]，那么grid\[0\]\[0\]应当是0，然后我们计算以\[0,0\]为起点的下一个能走到的位置集合，即\{\[2,1\],\[1,2\]\}，遍历这个集合，看看有没有哪一个位置的值是1，如果有，那么说明这一步是可以有效的走下去。现在起始位置变成了新位置，再继续上面这个处理。如此往复，直到遍历完所有位置。

思路和代码都不复杂，效率上嘛，也就是O(n^2)，考虑到问题难度是中等，我还隐约觉得会不会超出时间限制。代码提交以后，证明我想多了......

```java
	public boolean checkValidGrid(int[][] grid) {
		int n = grid.length;

		int i = 0, j = 0;
		if (grid[i][j] != 0) {
			return false;
		}

		for (int target = 1; target < n*n; target++) {
			List<int[]> nextPosList = nextPosList(i, j, n);
			if (nextPosList.isEmpty()) {
				return false;
			}

			int[] validNexPos = validNextPos(nextPosList, grid, target);
			if (validNexPos == null) {
				return false;
			}

			i = validNexPos[0];
			j = validNexPos[1];
		}

		return true;
	}

	private List<int[]> nextPosList(int i, int j, int n) {
		List<int[]> list = new ArrayList<>();

		int nextI = i + 2;
		int nextJ = j + 1;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}
		nextJ = j - 1;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}

		nextI = i + 1;
		nextJ = j + 2;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}
		nextJ = j - 2;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}

		nextI = i - 1;
		nextJ = j + 2;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}
		nextJ = j - 2;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}

		nextI = i - 2;
		nextJ = j + 1;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}
		nextJ = j - 1;
		if (validPos(nextI, nextJ, n)) {
			list.add(new int[] {nextI, nextJ});
		}

		return list;
	}

	private boolean validPos(int i, int j, int n) {
		if (i < 0 || i >= n) {
			return false;
		}
		if (j < 0 || j >= n) {
			return false;
		}
		return true;
	}

	private int[] validNextPos(List<int[]> nextPosList, int[][] grid, int target) {
		for (int[] nextPos : nextPosList) {
			int i = nextPos[0];
			int j = nextPos[1];

			if (grid[i][j] == target) {
				return nextPos;
			}
		}

		return null;
	}
```

# 问题1664 生成平衡数组的方案数 ★★☆☆☆
https://leetcode.cn/problems/ways-to-make-a-fair-array/

题目本身很容易理解，去掉数组中的一个元素，剩余元素按奇偶位置取和，取和相等的称为pair. 给定一个数组，计算有多少个位置满足pair。我往往是先从暴力解法入手，对于这个题目，暴力解法很简单，挨个位置遍历嘛，遍历每个位置时计算剩余元素的奇偶位置和。当然，不需要真的移动被去掉元素后面的元素的位置，因为很容易想明白，假设去掉某个位置的元素，后面元素向前补一位，那么这些补位的元素的位置的奇偶性会反转，即原来是奇数位置的会变成偶数位置，原来是偶数位置的会变成奇数位置。暴力解法的思路捋清楚以后，就要想如何优化，着手点就是看看哪里有重复计算。这个题目还是比较容易看出暴力解法的重复计算在哪的，暴力解法在遍历每个位置时都是独立的，没有利用上前序位置已经计算出的结果，即处理第i+1个位置时没有利用上第i个位置的计算结果。那么能不能利用上呢？我们定义四个变量，leftOddSum, leftEvenSum, rightOddSum, rightEvenSum，循环不变量是遍历到第i个位置时，leftOddSum和leftEvenSum表示第i个位置前（不包含第i个）的子序列分别的奇偶和，rightOddSum和rightEvenSum表示从第i个位置开始（包含第i个）的子序列分别的奇偶和。那么遍历到第i个位置表示要去除第i个元素，所以先根据第i个位置的奇偶性更新rightOddSum或rightEvenSum，然后比较leftOddSum和rightEvenSum的和是否与leftEvenSum和rightOddSum的和相等，如果相等，说明找到了一个pair位置。然后，在进入下一次循环前，将当前第i个位置的元素更新到leftOddSum或leftEvenSum.

```java
	public int waysToMakeFair(int[] nums) {
		int cnt = 0;

		int leftOddSum = 0, leftEvenSum = 0;
		int rightOddSum = 0, rightEvenSum = 0;
		for (int i = 0; i < nums.length; i++) {
			if (i % 2 == 0) {
				rightEvenSum += nums[i];
			} else {
				rightOddSum += nums[i];
			}
		}

		for (int i = 0; i < nums.length; i++) {
			boolean curIdxEven = i % 2 == 0;

			if (curIdxEven) {
				rightEvenSum -= nums[i];
			} else {
				rightOddSum -= nums[i];
			}

			if (leftEvenSum + rightOddSum == leftOddSum + rightEvenSum) {
				cnt++;
			}

			if (curIdxEven) {
				leftEvenSum += nums[i];
			} else {
				leftOddSum += nums[i];
			}
		}

		return cnt;
	}
```