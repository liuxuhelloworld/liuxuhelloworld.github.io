# 题目链接
https://leetcode-cn.com/problems/container-with-most-water/

# 解答过程
这个题目是挺久以前写的了，印象中当时也是参考官方题解写出来的。官方题解把解法归为双指针，我更愿意看作某种贪婪算法。为什么根据height[i]和height[j]的大小关系，舍弃小的那边就可以得到正确答案呢？这样想，每次根据containerWidth和containerHeight计算出的area，就是以height[i]或height[j]中较小的那根挡板作为容器的边之一，所能得到的最大area了。比如，假设height[i]小于height[j]，那么如果height[i]必须作为挡板之一，你已经计算出所能得到的最大area了，因为containerWidth是一直减小的，比height[i]更高的挡板没有用（因为height[i]本身是那个短板），比height[i]更低的只能进一步降低area。所以舍弃小的那边循环处理就能得到最终答案。

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
