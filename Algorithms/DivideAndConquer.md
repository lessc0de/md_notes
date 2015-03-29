# Divide and Conquer Paradigm

1. **DIVIDE** the problem into smaller sub-problems.
2. **CONQUER** each sub-problem via recursive calls.
3. Combine the solutions of sub-problems into one for the original problem.

Generally, the most amount of ingenuity occurs during the 3rd step. How do you combine solutions to sub-problems in the solution for the original problem?

## Example Problem
Input: array `A` containing numbers 1, 2, 3, ... in some arbitrary order.
Output: number of **inversions** = number of pairs `(i,j)` of array indices such that `i < j` and `A[i] > A[j]`.

### High-Level Algorithm
Call an inversion `(i, j)` with `i < j`:

* **left** if `i, j <= n / 2`
* **right** if `i, j > n / 2`
* **split** if `i <= n / 2 < j`

```
Count-Inversions (A)
  let n = len(A)
  
  if n == 1 return 0
  else
    x = Count-Inversions(A[:n/2+1])
    y = Count-Inversions(A[n/2+1:])
    z = Count-Split-Inversions(A)
```

If `Count-Split-Inversions` is implemented in `O(n)` time, then `Count-Inversions` will run in `O(n lg n)` time (just like merge sort).

### High-Level Algorithm (revised)
Merge Sort is sort of the proto-typical Divide & Conquer algorithm. Let's take a look at the merge step:

	let D[1..n] be an output, sorted array
    let B[1..n/2] be the 1st input, sorted array
    let C[1..n/2] be the 2nd input, sorted array
    i = 1, j = 1
    
    for k = 1 to n
      if B[i] < C[j]
        D[k] = B[i]
        i++
      else if C[j] < B[i]
        D[k] = C[j]
        j++

Suppose the input array `A` has no split inversions. What is the relationship between the sorted subarrays `B` and `C`? Well, all elements of `B` are less than all elements of `C`.

Therefore, during the merge step, whenever `C[j] < B[i]`, there is an inversion pair `i', j for i <= i' <= n/2`. We now have our algorithm for doing `Count-Split-Inversions`:

	Sort-And-Count-Inversions (A)
      let n = len(A)
      
      if n == 1 return 0
      else
        (B, x) = Sort-And-Count-Inversions(A[:n/2+1])
        (C, y) = Sort-And-Count-Inversions(A[n/2+1:])
        (D, z) = Merge-And-Count-Split-Inversions(A, B, C)
        return x + y + z
      
    Merge-And-Count-Split-Inversions (A, B, C)
      i = 1, j = 1
      z = 0
      
      for k = 1 to n
        if B[i] < C[j]
          D[k] = B[i]
          i++
        else if C[j] < B[i]  // inversion
          z += len(B[i:])
          D[k] = C[j]
          j++
      
      return D, z

# Master Theorem/Method
Potentially useful algorithmic ideas often need mathematical analysis to evaluate. Recall grade-school multiplication algorithm uses `O(n^2)` operations to multiply.

In the recursive approach for `x * y`...

* write `x = 10^(k/2) * a + b`
* write `y = 10^(k/2) * c + d`

where `k` is the number of digits in the number. Then:

	x * y = 10^n * ac + 10^(n/2) * (ad + bc) + bd    (*)

Just recursively compute `ac, ad, bc, bd` and compute `(*)` in the obvious way.

## Recurrence Analysis
Let `T(n)` be the maximum number of operations this algorithm needs to multiply two `k`-digit numbers. A recurrence expresses `T(k)` in terms of running time of recursive calls.

A recurrence has a base case and a recurrence case. The base case: `T(1) <= a constant`.

For all `k > 1`, `T(k) <= 4 * T(k/2) + O(k)`. `4T(k/2)` is the 4 recursive calls to multiplication. `O(k)` is the number of addition in each recursive step.

## A Better Recursive Algorithm
Recursively compute `ac, bd, (a+b)(c+d)`. Recall `ad + bc = (a+b)(c+d) - ac - bd`. So this is our new algorithm. What is our recurrence?

* `T(1) <= a constant`
* `for k > 1, T(k) = 3T(k/2) + O(k)`

## Formal Statement

1. Base Case: `T(n) <= a constant`, for all sufficiently small `n`
2. For all larger `n`: `T(n) <= a T(n/b) + O(n^d)`, where
  * `a` is the number of recursive calls at each recursive step (`>= 1`)
  * `b` is the input size shrinkage factor (`> 1`)
  * `d` is the complexity/exponent of the "combine step" (`>= 0`)

Then,

* if `a = b^d`, then `T(n) = O(n^d log n)`
* if `a < b^d`, then `T(n) = O(n^d)`
* if `a > b^d`, then `T(n) = O(n^(log_b a))`

