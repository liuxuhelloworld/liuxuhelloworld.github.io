<a id="problem_7"></a>
# 问题7 整数反转 ★☆☆☆☆
https://leetcode.cn/problems/reverse-integer/

按十进制位反转一个整数的思路是比较直接的，由低位到高位，我们可以很方便地遍历一个整数的十进制位，在遍历过程中，也很容易维护一个反转后的整数值。唯一的难点是如何处理overflow的问题。如果可以使用长整型，那么overflow的问题也很容易解决。但是题目明确要求不要使用长整型。稍作思考，只能在可能overflow前去判断。

```java
	public int reverse(int x) {
		if (x == Integer.MIN_VALUE) {
			return 0;
		}

		int maxIntDivide10 = Integer.MAX_VALUE / 10;
		int maxIntDivide10Remain = Integer.MAX_VALUE % 10;

		boolean negative = x < 0;
		int absX = x >= 0 ? x : -x;

		int reverse = 0;
		while (absX > 0) {
			int remain = absX % 10;

			if (reverse > maxIntDivide10
				|| (reverse == maxIntDivide10 && remain > maxIntDivide10Remain)) {
				return 0;
			}

			reverse = reverse * 10 + remain;

			absX = absX / 10;
		}

		return negative ? -reverse : reverse;
	}
```

<a id="problem_8"></a>
# 问题8 字符串转换整数 ★☆☆☆☆
https://leetcode-cn.com/problems/string-to-integer-atoi/

这个题目是经典的字符串转整数，从解题思路上来说，并不复杂，根据题目要求按顺序处理字符串就可以，需要注意的是如何处理overflow的问题。相比[问题7](#problem_7)，这个题目并没有要求不能使用长整型，所以偷懒的做法就是直接用长整型。如果题目要求不能使用长整型，那么就需要使用和问题7中类似的方法，在overflow之前去判断，特别的，需要考虑到32位整数取值范围的非对称性。

```java
    public int myAtoi(String s) {
        char[] chars = s.toCharArray();

        int i = 0;
        while (i < chars.length && Character.isSpaceChar(chars[i])) {
            i++;
        }

        if (i == chars.length
            || (chars[i] != '+' && chars[i] != '-' && !Character.isDigit(chars[i]))) {
            return 0;
        }

        boolean negative = false;
        if (chars[i] == '+' || chars[i] == '-') {
            negative = chars[i] == '-';
            i++;
        }

        long ret = 0;
        while (i < chars.length && Character.isDigit(chars[i])) {
            ret = ret * 10 + (chars[i]-'0');
            if (ret > Integer.MAX_VALUE) {
                break;
            }
            i++;
        }
        ret = ret * (negative ? -1 : 1);

        if (ret > Integer.MAX_VALUE) {
            return Integer.MAX_VALUE;
        } else if (ret < Integer.MIN_VALUE) {
            return Integer.MIN_VALUE;
        } else {
            return (int) ret;
        }
    }
```

# 问题12 整数转罗马数字 ★☆☆☆☆
https://leetcode-cn.com/problems/integer-to-roman/

题目乍一看有点唬人，简单来说，就是将阿拉伯数字转换为罗马数字。那么很直接的，按照罗马数字的进制规则，由大到小的处理基数就可以了，即按照1000、900、500、400、100、90、50、40、10、9、5、4、1的顺序，分别做如下计算：用当前值和基数做整数除法，计算得出包含的基数个数，添加到罗马数字表达法中，然后用余数和下一个基数重复此过程。

```java
	private static int[] val = {1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1};
	private static String[] symbol = {"M", "CM", "D", "CD", "C", "XC", "L", "XL", "X", "IX", "V", "IV", "I"};

	public String intToRoman(int num) {
		StringBuilder builder = new StringBuilder();

		int i = 0;
		while (num != 0) {
			int quotient = num / val[i];
			if (quotient != 0) {
				for (int j = 0; j < quotient; j++) {
					builder.append(symbol[i]);
				}
			}
			num = num % val[i];
			i++;
		}

		return builder.toString();
	}
```

# 问题1362 最接近的因数 ★☆☆☆☆
https://leetcode-cn.com/problems/closest-divisors/

这个题目我想到的就是暴力解法，抽象出一个计算某个数绝对值差最小的乘数对的方法，然后分别计算num+1和num+2的绝对值差最小的乘数对，然后比较它们哪个绝对值差更小。提交以后是没有问题的，不过效率是差了点。额，看了下官方题解，基本是一样的思路。

```java
	public int[] closestDivisors(int num) {
		int target1 = num + 1;
		int target2 = num + 2;

		int[] divisors1 = minDiffDivisors(target1);
		int[] divisors2 = minDiffDivisors(target2);

		if (Math.abs(divisors1[0] - divisors1[1]) < Math.abs(divisors2[0] - divisors2[1])) {
			return divisors1;
		} else {
			return divisors2;
		}
	}

	private int[] minDiffDivisors(int num) {
		int max = (int) Math.floor(Math.sqrt(num));

		int minDiff = Integer.MAX_VALUE;
		int[] ret = new int[2];
		for (int i = 1; i <= max; i++) {
			if (num % i == 0) {
				int j = num / i;
				int curDiff = Math.abs(i - j);
				if (curDiff < minDiff) {
					minDiff = curDiff;
					ret[0] = i;
					ret[1] = j;
				}
			}
		}

		return ret;
	}
```

# 问题2481 分割圆的最少切割次数 ★☆☆☆☆
https://leetcode.cn/problems/minimum-cuts-to-divide-a-circle/

这个题目有点意思，我觉得是一个不错的思维题。

那么按照题目要求的将一个圆n等分，至少需要的cut数是多少呢？将一个圆n等分的结果是以圆心为顶点的n个角，每个角大小一样。每个角两条边，但是相邻的两个角共用一条边，所以将一个圆n等分，n条边肯定是够了，也就是n个cut。这个题目的关键就在于，这n个半长的cut中如果有2条可以连成1条全长的cut，就等于减小了一个cut，也就是说可能不需要n条边那么多就能将一个圆n等分。

那么什么情况下可以2条半长的cut连成1条全长的cut，从而减小cut数呢？这个地方我一开始陷入了一个错误的思路，我的直觉是需要考虑n等分后1个角的大小是否能整除180度，比如，将一个圆6等分，6条半长的cut肯定够了，但是180/60=3，所以只需要3条全长的cut就够了。然后我就提交了一版答案，先判断n是否能整除360，能整除的，再计算一个角的大小，再判断一个角的大小能否整除180。结果题目判定失败，错误示例，14等分需要的最小cut数是7，而不是14。

然后我就陷入了迷惑，14不能整除360，14等分后的一个角的大小也不能整除180，怎么会有能连成全长cut的呢？再稍作思考，才发现问题所在。关键不在于是不是能整除180，即便14等分的一个角不能整除180，但无论如何，14等分的一半是不是半圆？既然是半圆，那么显然任意连续的七个14等分角就都是半圆，所以最小cut数是7，而不是14。由此可推，是否能将2个半长的cut连成1个全长的cut，只取决于n是否能被2整除，只要能被2整除，那么就可以减半cut数。所以，答案其实很简单，判断n是否能被2整除，能整除的需要n/2个cut，不能整除的需要n个cut。

为什么我一开始会陷入认为需要考虑单角能否整除180的错误里呢？归根结底我觉得是对无理数、对数的连续性缺乏认知。由此我还想到数学史上的一桩轶事，就是如何只用直尺和圆规将一个圆17等分。据说这是伟大的数学家高斯错拿了老师的研究题目，而花了一夜时间解决了这个问题，第二天和老师抱怨昨天的题目太难了，老师看了以后惊掉下巴......

```java
	public int numberOfCuts(int n) {
		if (n == 1) {
			return 0;
		}

		if (n % 2 == 0) {
			return n / 2;
		} else {
			return n;
		}
	}
```

# 问题1015 可被K整除的最小整数 ★★☆☆☆
https://leetcode.cn/problems/smallest-integer-divisible-by-k/

这个题目算是比较纯粹的数学题目了。题目本身不难理解，给定一个整数k，求完全由数字1组成的、能被k整除的、最小的那个整数包含的数字1的个数。

从简单直接的想法入手，我们可以由小到大挨个遍历1、11、111、...，直到遍历到能被k整数的那一个。但是这里面有两个问题：
- 题目明确说明不限于64位整数的范围，遍历一定次数后会overflow
- 如果对于给定的k，不存在满足题目要求的整数呢？也就是什么时候跳出遍历
  
对于第一个问题，某些语言可能能支持大数，比如Java中使用BigInteger，但这样未免过于取巧。那么应当如何避免溢出问题呢？对于题目而言，其实我们关心的是当前遍历到的完全由数字1组成的整数对k取余的结果，当取余为0时，说明我们找到了满足题目要求的整数。换句话说，我们不需要真的由小到大遍历每个完全由数字1组成的整数，而只需要维护这样的整数对k取余的结果即可。而根据数学运算，x1 % k == (x * 10 + 1) % k == (x % k) * 10 + 1

对于第二个问题，直接说答案吧，如果对于给定的k，不存在满足题目要求的整数，那么由小到大遍历完全由数字1组成的整数，并对k取余，这个余数一定是1到k-1中的一个，并且会按一定顺序挨个出现，直到重复。额，具体的数学推导我是不记得了，但这个结论还是有印象的，应该是和费马大定理有关。

另外，对于尾数是偶数或者是5的，可以直接返回，肯定不会存在满足题目要求的整数。

```java
	public int smallestRepunitDivByK(int k) {
		if (k == 1) {
			return 1;
		}

		if (k % 2 == 0) {
			return -1;
		}

		if (k % 5 == 0) {
			return -1;
		}

		int cnt = 1;
		int val = 1;
		while (true) {
			val = val*10 + 1;
			cnt++;

			int remainder = val % k;
			if (remainder % k == 0) {
				return cnt;
			}

			if (remainder == 1) {
				break;
			}

			val = remainder;
		}

		return -1;
	}
```