# 题目链接
https://leetcode-cn.com/problems/unique-binary-search-trees/

# 解答过程
这个题目我觉得就很像智力题了，或者说纯算法题，如果纠结于具体的BST形态就想错方向了。将从1到n的n个节点排布成BST，重点是将问题抽象，从整棵树的角度考虑。固定一个节点k作为跟节点，所有小于跟节点k的节点组成左子树，所有大于跟节点k的节点组成右子树，那么以k为跟节点的BST有多少个？显然是S(k-1)\*S(n-k)。k的取值有从1到n的n种可能，所以：

```java
	public int numTrees(int n) {
		assert n >= 1 && n <= 19;

		int[] num = new int[n+1];
		num[0] = 1;

		for (int i = 1; i <= n; i++) {
			for (int j = 0; j <= i-1; j++) {
				num[i] += num[j] * num[i-1-j];
			}
		}

		return num[n];
	}
```

我印象中这个数在数学中是有专门的名称的，叫卡塔尔数，有时间真想重学高等数学。