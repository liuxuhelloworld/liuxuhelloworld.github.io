# 题目链接
https://leetcode.cn/problems/serialize-and-deserialize-bst/

# 解答过程
题目要求很好理解了，简单来说，给定一颗二叉查找树，写两个方法，一个方法可以把二叉查找树序列化为字符串，另一个方法可以把序列化的字符串反序列化为同样结构的二叉查找树。

我是从反序列化方法入手的，先搞清楚如何从一个字符串能构建出一颗二叉查找树，这里面隐含着我们需要怎样的一个字符串，这个字符串需要让我们比较方便的重建二叉查找树。那么我们需要怎样的一个字符串呢？这就要从二叉查找树这个特性入手了，因为单纯的一个字符串直觉上很难包含树结构信息。这个地方稍作思考，可以发现，如果我们有二叉查找树先序遍历的结果，那么就可以比较方便的构造，逻辑如下：第一个元素显然是根节点，剩下的元素可以一分为二，左边的是左子树，右边的是右子树，而继续构造左子树或右子树的过程也是如此，利用递归很容易实现这个逻辑。OK，通过先序遍历的结果可以重构二叉查找树了，那么如果通过字符串得到先序遍历的结果呢？或者反过来说，如果通过先序遍历的结果得到一个字符串呢？这种转换显然是可以相互的。这个地方再稍作思考，比较简单直接的方法就是把先序遍历的结果以数组的形式存储，然后把这个数组转换为字符串。一个数组就是连续的字节序列，转换为字符串的简单方法就是每四位比特为一组解释为十六进制字符串。

基于以上思路，代码如下：
```java
	public String serialize(TreeNode root) {
		if (root == null) {
			return null;
		}

		List<Integer> inOrderList = new ArrayList<>();
		inOrderTraverse(root, inOrderList);

		StringBuilder builder = new StringBuilder();
		for (int i = 0; i < inOrderList.size(); i++) {
			short cur = inOrderList.get(i).shortValue();

			int val = (cur & 0b1111000000000000) >>> 12;
			char ch = map(val);
			builder.append(ch);

			val = (cur & 0b0000111100000000) >>> 8;
			ch = map(val);
			builder.append(ch);

			val = (cur & 0b0000000011110000) >>> 4;
			ch = map(val);
			builder.append(ch);

			val = cur & 0b0000000000001111;
			ch = map(val);
			builder.append(ch);
		}

		return builder.toString();
	}

	private void inOrderTraverse(TreeNode node, List<Integer> inOrderList) {
		if (node == null) {
			return;
		}

		inOrderList.add(node.val);
		inOrderTraverse(node.left, inOrderList);
		inOrderTraverse(node.right, inOrderList);
	}

	private char map(int val) {
		char ch = 0;
		switch (val) {
			case 0:
				ch = '0';
				break;
			case 1:
				ch = '1';
				break;
			case 2:
				ch = '2';
				break;
			case 3:
				ch = '3';
				break;
			case 4:
				ch = '4';
				break;
			case 5:
				ch = '5';
				break;
			case 6:
				ch = '6';
				break;
			case 7:
				ch = '7';
				break;
			case 8:
				ch = '8';
				break;
			case 9:
				ch = '9';
				break;
			case 10:
				ch = 'A';
				break;
			case 11:
				ch = 'B';
				break;
			case 12:
				ch = 'C';
				break;
			case 13:
				ch = 'D';
				break;
			case 14:
				ch = 'E';
				break;
			case 15:
				ch = 'F';
				break;
			default:
				break;
		}

		return ch;
	}

	public TreeNode deserialize(String data) {
		if (data == null
			|| data.isEmpty()) {
			return null;
		}

		char[] chars = data.toCharArray();
		int length = chars.length;

		short[] inOrderArray = new short[length / 4];

		for (int i = 0; i < inOrderArray.length; i++) {
			char ch1 = chars[i*4];
			char ch2 = chars[i*4 + 1];
			char ch3 = chars[i*4 + 2];
			char ch4 = chars[i*4 + 3];

			int cur = 0;

			int val = map(ch1);
			cur = cur | (val << 12);

			val = map(ch2);
			cur = cur | (val << 8);

			val = map(ch3);
			cur = cur | (val << 4);

			val = map(ch4);
			cur = cur | val;

			inOrderArray[i] = (short) cur;
		}

		return rebuild(inOrderArray, 0, inOrderArray.length - 1);
	}

	private int map(char ch) {
		int val = 0;
		switch (ch) {
			case '0':
				val = 0;
				break;
			case '1':
				val = 1;
				break;
			case '2':
				val = 2;
				break;
			case '3':
				val = 3;
				break;
			case '4':
				val = 4;
				break;
			case '5':
				val = 5;
				break;
			case '6':
				val = 6;
				break;
			case '7':
				val = 7;
				break;
			case '8':
				val = 8;
				break;
			case '9':
				val = 9;
				break;
			case 'A':
				val = 10;
				break;
			case 'B':
				val = 11;
				break;
			case 'C':
				val = 12;
				break;
			case 'D':
				val = 13;
				break;
			case 'E':
				val = 14;
				break;
			case 'F':
				val = 15;
				break;
			default:
				break;
		}

		return val;
	}

	private TreeNode rebuild(short[] inOrderArray, int start, int end) {
		if (start > end) {
			return null;
		}

		short val = inOrderArray[start];
		TreeNode node = new TreeNode(val);

		int lessThan = -1;
		int greaterThan = -1;
		for (int i = start + 1; i <= end; i++) {
			if (inOrderArray[i] < val) {
				lessThan = i;
			} else if (inOrderArray[i] > val) {
				greaterThan = i;
				break;
			}
		}

		if (lessThan != -1) {
			node.left = rebuild(inOrderArray, start + 1, lessThan);
		}
		if (greaterThan != -1) {
			node.right = rebuild(inOrderArray, greaterThan, end);
		}

		return node;
	}
```

代码提交以后是没有问题的，但是感觉代码有点复杂。看了一下官方题解，思路是差不多的，不过这个直接把遍历结果toString的处理还是有点，额，序列化的结果未免看着太不专业了吧......