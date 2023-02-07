# 题目链接
https://leetcode.cn/problems/alert-using-same-key-card-three-or-more-times-in-a-one-hour-period/

# 解答过程
这个题目从思路来说我觉得并不难，属于比较侧重代码书写的题目。按用户归集打卡时间后遍历，遍历过程中先排序，然后判断是否存在一个小时内三次或以上打开记录，这里可以使用滑动窗口的思想来提高效率。但是转念想，因为目标是三次或以上，是不是直接隔一个元素判断就可以？

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
		int windowCnt = 0;
		int start = 0, end = 0;
		while (end < timeArray.length) {
			String startTime = timeArray[start];
			String endTime = timeArray[end];
			if (isOneHourPeriod(startTime, endTime)) {
				windowCnt++;
				if (windowCnt >= 3) {
					return true;
				}
				end++;
			} else {
				windowCnt--;
				start++;
			}
		}

		return false;
	}

	private boolean isOneHourPeriod(String time1, String time2) {
		int hour1 = Integer.valueOf(time1.substring(0, 2));
		int minute1 = Integer.valueOf(time1.substring(3, 5));
		int hour2 = Integer.valueOf(time2.substring(0, 2));
		int minute2 = Integer.valueOf(time2.substring(3, 5));

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

额，看了一下官方题解，果然直接隔一个元素直接比较就可以了......