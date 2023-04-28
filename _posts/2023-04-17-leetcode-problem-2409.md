# 题目链接
https://leetcode.cn/problems/count-days-spent-together/

# 解答过程
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