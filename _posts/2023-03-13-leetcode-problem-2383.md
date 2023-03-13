# 题目链接
https://leetcode.cn/problems/minimum-hours-of-training-to-win-a-competition/

# 解答过程
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