# 题目链接
https://leetcode-cn.com/problems/container-with-most-water/

# 解答过程
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

# 相似题目
- [Problem45-Jump Game](2022-05-07-leetcode-problem-45.md)
- [Problem55-Jump Game](2022-05-09-leetcode-problem-55.md)
- [Problem2178-Maximum Split of Positive Even Integers](2023-07-10-leetcode-problem-2178.md)