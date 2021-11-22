# 2.1-1
Using Figure 2.2 as a model, illustrate the operation of INSERTION-SORT on the array A = {31, 41, 59, 26, 41, 58}.

**Solution:**
todo

# 2.1-2
Rewrite the INSERTION-SORT procedure to sort into nonincreasing instead of nondecreasing order. 

**Solution:**

```
NON-INCREASING-INSERTION-SORT(A)
	for j = 2 to A.length
		key = A[j]
		// insert A[j] into the reversed sorted sequence A[1..j-1]
		i = j - 1
		while i > 0 and A[i] < key
			A[i+1] = A[i]
			i = i - 1
		A[i+1] = key
```

# 2.1-3
Consider the searching problem:
Input: A sequence of n numbers A = <a1, a2, ... an> and a value v.
Output: An index i such that v = A[i] or the special value NIL if v does not appear in A.
Write pseudocode for linear search, which scans through the sequence, looking for v. Using a loop invariant, prove that your algorithm is correct. Make sure that your loop invariant fulfills the three necessary properties.  

**Solution:**

```
LINEAR-SEARCH(A, v)
		for i = 1 to A.length
				if A[i] == v
						return i
		return NIL
```

loop invariant: 每次循环时，A[1..i-1]中不包含v

1. initialization: 第1次循环时，i = 1，A[1..i-1]为空，不包含v

2. maintenance: 第i次循环时，如果A[i]等于v，则结束，不会进入第i+1次循环；如果A[i]不等于v，则进入第i+1次循环时，A[1..i]中不包含v

3. termination: 循环结束时，i等于A.length+1，所以A[1..A.length]中不包含v

# 2.1-4

Consider the problem of adding two n-bit binary integers, stored in two n-element arrays A and B. The sum of the two integers should be stored in binary form in an (n+1)-element array C . State the problem formally and write pseudocode for adding the two integers.  

**Solution:**

Input: two n-element sequence of A = < $a_1$ , $a_2$ , $a_3$ , ... , $a_n$ > and B = < $b_1$ , $b_2$ , $b_3$ ,  ..., $b_n$ >, each represents a n-bit binary integer, $a_i,b_i \in \{0, 1\}$ for $1<=i<=n$

Output: a (n+1)-element sequence of C = <$c_1$ , $c_2$ , ..., $c_{n+1}$ >, such C = A+B

```
SUM(A, B)
    carry = 0
    for i = A.length to 1
        C[i+1] = (A[i] + B[i] + carry) % 2
        carry = (A[i] + B[i] + carry) / 2
    C[1] = carry
    return C
```