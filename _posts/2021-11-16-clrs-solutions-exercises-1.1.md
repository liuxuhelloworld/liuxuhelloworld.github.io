# 1.1-1

Give a real-world example that requires sorting or a real-world example that requires computing a convex hull.

**Solution:** 

排序的场景很多，比如，游戏中的玩家排名

# 1.1-2

Other than speed, what other measures of efficiency might one use in a real-world setting?

**Solution:** 

memory space

# 1.1-3

Select a data structure that you have seen previously, and discuss its strengths and limitations.

**Solution:** 

binary search tree. 一般情况下，可以高效地支持ordered symbol table操作。缺点是对输入敏感，bad input会导致树的形态变差，效率降低。

# 1.1-4

How are the shortest-path and traveling-salesman problems given above similar? How are they different?

**Solution:**

shortest-path问题是给定两点，找出连通它们的代价最小的路径。

traveling-salesman问题是从某一点出发，遍历其他所有点后回到原出发点，找出代价最小的路径。

它们都基于图来建模，目标都是寻找代价最小的路径，但是明显traveling-salesman问题更复杂。

# 1.1-5

Come up with a real-world problem in which only the best solution will do. Then come up with one in which a solution that is “approximately” the best is good enough.

**Solution:**

在数值计算中，应该有很多只需要approximately the best的情形？