# 2.2-1
Express the function $n^3/1000 - 100n^2 - 100n + 3$ in terms of $\Theta$-notation.

**Solution**: $\Theta$($n^3$)

# 2.2-2
Consider sorting n numbers stored in array A by first finding the smallest element of A and exchanging it with the element in A\[1\]. Then find the second smallest element of A, and exchange it with A\[2\]. Continue in this manner for the first n-1 elements of A. Write pseudocode for this algorithm, which is known as **selection sort**. What loop invariant does this algorithm maintain? Why does it need to run for only the first n-1 elements, rather than for all n elements? Give the best-case and worst-case running times of selection sort in $\Theta$ notation.

**Solution**:
```
SELECTION-SORT(A)
  for i = 1 to A.length-1
    min = i
    // find the smallest in sequence A[i..n]
    for j = i+1 to A.length
      if A[j] < A[min]
        min = j
    tmp = A[i]
    A[i] = A[min]
    A[k] = tmp
```

loop invariant: A[1..i-1] is ordered before ith iteration, and no element larger then A[i..n]

best-case running time: $\Theta$($n^2$)

worst-case running time: $\Theta$($n^2$)

# 2.2-3
Consider linear search again (see Exercise 2.1-3). How many elements of the input sequence need to be checked on the average, assuming that the element being searched for is equally likely to be any element in the array? How about in the worst case? What are the average-case and worst-case running times of linear search in $\Theta$ notation? Justify your answers.

**Solution**: 
$n/2$ elements need to be checked on the average.

$n$ elements need to be checked in the worst case.

average-case running time: $\Theta$($n$)

worst-case running time: $\Theta$($n$)

# 2.2-4
How can we modify almost any algorithm to have a good best-case running time?

**Solution**: a special case test