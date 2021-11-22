# 1.2-1

Give an example of an application that requires algorithmic content at the application level, and discuss the function of the algorithms involved.

**Solution:**

实时地图导航，根据起始位置、交通状况实时计算出导航路径。

# 1.2-2

Suppose we are comparing implementations of insertion sort and merge sort on the same machine. For inputs of size n, insertion sort runs in $8n^2$ steps, while merge sort runs in $64nlgn$ steps. For which values of n does insertion sort beat merge sort?

**Solution:**
$$
8n^2 < 64nlgn \\
n < 8lgn
$$

计算$f(x) = 8lgx - x$的解即可，用二分法简单写了下，求解出$x\approx43.559$，即当n小于等于43时，insertion sort beat merge sort.

```java
    private static final double EPS = 0.0001;

    private static double f(double x) {
        return 8*(Math.log(x)/Math.log(2)) - x;
    }

    public static void main(String[] args) {
        double left, right, mid;

        left = 1; right = 100;
        while (true) {
            mid = (left + right) / 2;

            if (Math.abs( f(mid)) <= EPS) {
                break;
            }

            if (f(mid) > 0) {
                left = mid;
            } else if (f(mid) < 0) {
                right = mid;
            }
        }
        System.out.println(mid);
    }
```

# 1.2-3

What is the smallest value of n such that an algorithm whose running time is $100n^2$ runs faster than an algorithm whose running time is $2^n$ on the same machine?
$$
100n^2 < 2^n \\
$$
类似的，计算$f(x)=100n^2 - 2^n$的解即可，求解出$x\approx14.324$，即n最小为15

```java
    private static final double EPS = 0.0001;

    private static double f(double x) {
        return 100*x*x - Math.pow(2, x);
    }

    public static void main(String[] args) {
        double left, right, mid;

        left = 1; right = 20;
        while (true) {
            mid = (left + right) / 2;
            if (Math.abs( f(mid)) <= EPS) {
                break;
            }
            if (f(mid) > 0) {
                left = mid;
            } else if (f(mid) < 0) {
                right = mid;
            }
        }
        System.out.println(mid);
    }
```