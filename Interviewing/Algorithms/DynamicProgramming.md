# Dynamic Programming
## Summary
Dynamic Programming applies when the problem has these characteristics:

### Recursive Decomposition
The problem has recursive structure: it breaks down into smaller problems of the same type (**subproblem**). _This characteristic is shared with divide and conquer, but dynamic programming is distinguished by the next item._

### Overlapping Subproblems
The subproblems solved by a recursive solution overlap (the same subproblems are revisited more than once). _This means we can save time by preventing the redundant computations._

### Optimal Substructure
Any optimal solution involves making a choice that leaves one or more subproblems to solve, and the solutions to the subproblems used within the optimal solution must themselves be optimal. _This means that optimized recursive solutions can be used to construct optimized larger solutions._

## A Problem Solving Method
We problem solve with dynamic programming in four steps:

1. **Characterize the structure of an optimal solution.**
  * How are optimal solutions composed of optimal solutions to subproblems?
2. **Recursively define the value of an optimal solution**
  * Write a recursive cost function that reflects the above structure
3. **Compute the value of an optimal solution**
  * Write code to compute the recursive values, memoizing or solving smaller problems first to avoid redundant computation
4. **Construct an optimal solution from the computed information.**
  * Augment the code as needed to record the structure of the solution.

## Complexity Analysis
<table>
<tr>
  <th>
  </th>
  <th>
# subproblems in optimal solution
  </th>
  <th>
# choices to consider
  </th>
  <th>
# subproblems overall
  </th>
  <th>
overall run-time
  </th>
</tr>
<tr>
  <td>
rod cutting
  </td>
  <td>
<code>1 (size n-i)</code>
  </td>
  <td>
<code>n (look at each i)</code>
  </td>
  <td>
<code>O(n)</code>
  </td>
  <td>
<code>O(n^2)</code>
  </td>
</tr>
<tr>
  <td>
optimal binary search tree
  </td>
  <td>
<code>2 (one size i, one size n-i)</code>
  </td>
  <td>
<code>j - i + 1</code>
  </td>
  <td>
<code>O(n^2)</code>
  </td>
  <td>
<code>O(n^3)</code>
  </td>
</tr>
<tr>
  <td>
max profit in <code>k</code> trades
  </td>
  <td>
<code>2</code>
  </td>
  <td>
<code>O(n)</code>
  </td>
  <td>
<code>k</code>
  </td>
  <td>
<code>O(kn)</code>
  </td>
</tr>
</table>

## Example 1: Rod Cutting
Input: a table of prices for each rod length.  
Output: maximum revenue for rods of lengths summing to `n`.

### 1. Characterize the structure of an optimal solution.
Suppose you have a rod of some length, and you determine (somehow) that the optimal solution involves a cut somewhere on the rod. It must be the case that the optimal solution to the original problem incorporates optimal solutions to the original problem's subproblems. A subproblem is the original problem but with smaller input length (`n`).

### 2. Recursively define the value of an optimal solution.
Let's write an equation that tells us about this optimal substructure.
	
    r = revenue[]
    p = price[]
	
    r[n] = max(p[n], r[1] + r[n-1], r[2] + r[n-2], ..., r[n-1] + r[1])
         = max(p[n], p[1] + r[n-1], p[2] + r[n-2], ..., p[n-1] + r[1])
         = max(p[i] + r[n-i]) for 1 <= i <= n

	
    CUT-ROD (p, n)
      if n == 0
        return 0
      q = -oo
      for i = 1 to n
        q = max(q, p[i] + CUT-ROD(p, n-i))
      return q
    
### 3. Compute the value of an optimal solution
We want to make the computation more efficient with memoization. Notice that some subproblems are solved multiple times. Let's store those results into an array and reference the array when the same subproblem is later encountered.

	MEMOIZED-CUT-ROD (p, n)
      let r[0..n] be a new array
      r[0] = 0
      for i = 1 to n
        r[i] = -oo
      return MEMOIZED-CUT-ROD-AUX(p, n, r)
    
	MEMOIZED-CUT-ROD-AUX (p, n, r)
      if r[n] > -oo
        return r[n]
      q = -oo
      for i = 1 to n
        q = max(q, p[i] + MEMOIZED-CUT-ROD-AUX(p, n-i, r))
      r[n] = q
      return q

This is top-down memoization. A bottom-up strategy is less expensive because it doesn't need a recursive stack.

	BOTTOM-UP-CUT-ROD (p, n)
      let r[0..n] be a new array
      r[0] = 0
      for i = 1 to n
        q = -oo
        for j = 1 to i
          q = max(q, p[j] + r[i - j])
        r[j] = q
      return r[n]
        
### 4. Construct an optimal solution from the computed information.
The above programs return the value of an optimal solution. To construct the solution itself, we need to record the choices that led to the optimal solutions. Use a table `s` to record the place where the optimal cut was made (compare to `BOTTOM-UP-CUT-ROD`):

	EXTENDED-BOTTOM-UP-CUT-ROD (p, n)
      let r[0..n] and s[0..n] be new arrays
      r[0] = 0
      for i = 1 to n
        q = -oo
        for j = 1 to i
          if q < p[j] + r[i - j]
            q = p[j] + r[i - j]
            s[i] = j
        r[j] = q
      return r and s

After getting `r` and `s`, we then trace the choices made back through the table `s` with this procedure:

	PRINT-CUT-ROD-SOLUTION (p, n)
      (r, s) = EXTENDED-BOTTOM-UP-CUT-ROD(p, n)
      while n > 0
        print s[n]
        n -= s[n]


## Example 2: Maximum Profit
Input: a log of a stock's price for each day
Output: maximum profit after trading `k` times.

### 1. Characterize the structure of an optimal solution.
Suppose you determine (somehow) that the optimal solution involves a buy on some day and a sell on a later day. It must be the case that the optimal solution incorporates optimal solutions to the original problem's subproblems, i.e. it incorporates an optimal buy on some earlier day and an optimal sell afterwards.

A buy occurs after a sell, and a sell occurs after a buy. Therefore, an optimal buy in the original problem incorporates an optimal sell in some earlier day.

Also, an optimal sell in the original problem incorporates an optimal buy in some earlier day.

### 2. Recursively define the value of an optimal solution.

	i = day, 0 <= i <= n
    j = jth buy/sell, 0 <= j <= k
    p = price
    r = revenue
    
    b[n,k] = max(s[n-1, k-1] - p[n], s[n-2, k-1] - p[n], ..., s[2(k-1)+1, k-1] - p[n], s[2(k-1), k-1] - p[n])
   	       = max(s[n-i, k-1] - p[n]) for 1 <= i <= 2(k-1)
    s[n,k] = max(b[n-1, k] + p[n], b[n-2, k] + p[n], ..., b[2(k-1), k] + p[n], b[2(k-1)-1, k] + p[n])
           = max(b[n-i, k] + p[n]) for 1 <= i <= 2(k-1)-1
    
    b[1,1] = -p[1]
    b[n,1] = max(b[n-1, 1], -p[n])
    
    
    PROFIT-SELL (p, n, k)
      q = -oo
      for i = 1 to 2(k-1)
        q = max(q, PROFIT-BUY(p, n-i, k))
      return q
    
    PROFIT-BUY (p, n, k)
      if n == 1 and k == 1
        return -p[1]
      if k == 1
        return max(PROFIT-BUY(p, n-1, 1), -p[n])
      q = -oo
      for i = 1 to 2(k-1)-1
        q = max(q, PROFIT-SELL(p, n-i, k-1))
      return q

### 3. Compute the value of an optimal solution
We want to make the computation more efficient with memoization.

	MAX-K-PAIRS-PROFITS (p, n, k)
      let s[1..n, 1..k] be a new array
      let b[1..n, 1..k] be a new array
      for i = 1 to n
        for j = 1 to k
          s[i, j] = -oo
          b[i, j] = -oo
      return MEMOIZED-PROFIT-SELL-AUX (p, n, k, s, b)
    
    MEMOIZED-PROFIT-SELL-AUX (p, n, k, s, b)
      if s[n, k] > -oo
        return s[n, k]
      q = -oo
      for i = 1 to s(2k-1)
        q = max(q, MEMOIZED-PROFIT-BUY-AUX(p, n-i, k, s, b)
      s[n, k] = q
      return q
    
    MEMOIZED-PROFIT-BUY-AUX (p, n, k, s, b)
      if b[n, k] > -oo
        return b[n, k]
      if n == 1 and k == 1
        q = -p[1]
      else if k == 1
        q = max(MEMOIZED-PROFIT-BUY-AUX(p, n-1, 1, s, b), -p[n])
      else
        q = -oo
        for i = 1 to 2(k-1)-1
          q = max(q, MEMOIZED-PROFIT-SELL-AUX(p, n-i, k-1, s, b)
      b[n, k] = q
      return q

Notice that some problems are re-solved at most one more time. There isn't much speed up top-down wise, but if we approach it bottom-up, not only do we eliminate the need for recursive calls, but we also get to use a different kind of memoization array and re-use it over and over again, thus saving us space complexity.

	BOTTOM-UP-MAX-K-PAIRS-PROFITS (p, n, k)
      let r[0..n, 0..2k] be a new array
      r[0, :] = -oo
      r[:, 0] = -oo
      for i = 1 to n
        for j = 1 to min(i, 2k)
          if j & 1  // if buy
            q = max((j == 1 ? 0 : r[i-1, j-1]) - p[i], r[i-1, j])
          else  // if sell
            q = max(r[i-1, j-1] + p[i], r[i-1, j])
          r[i, j] = q
      return r[n, 2k]

	BOTTOM-UP-EFFICIENT-MAX-K-PAIRS-PROFITS (p, n, k)
      let r[1..2k] be a new array
      r[:] = -oo
      for i = 1 to n
        let pre_r[1..2k] be a new array
        pre_r = CLONE(r)
        for j = 1 to min(i, 2k)
          if j & 1  // if buy
            q = max((j == 1 ? 0 : pre_r[j-1]) - p[i], pre_r[j])
          else  // if sell
            q = max(pre_r[j-1] + p[i], pre_r[j])
          r[j] = q
      return r[k]

### 4. Construct an optimal solution from the computed information.
Screw this step.


## Example 2: Longest Common Subsequence
Input: two strings a, b of lengths m, n respectively  
Output: the largest common substring (LCS) in a & b, and its length

### 1. Characterize the structure of an optimal solution.

```
let i be the char index of a
let j be the char index of b
let l[i,j] be the length of the LCS between a[:i+1] and b[:j+1]

l[i,0] = 0
l[0,j] = 0
if a[i] == b[j]
  l[i,j] = 1 + l[i-1, j-1]
else
  l[i,j] = max(l[i-1, j], l[i, j-1])
```

### 2. Recursively define the value of an optimal solution.

```
LCS (a, b)
  m = len(a), n = len(b)
  return LCS-HELPER(a, b, m, n)

LCS-HELPER (a, b, i, j)
  if i == 0 || j == 0
    return 0
  if a[i] == b[j]
    return 1 + LCS-HELPER(a, b, i-1, j-1)
  else
    return max(LCS-HELPER(a, b, i-1, j), LCS-HELPER(a, b, i, j-1))
```

### 3. Compute the value of an optimal solution

```
LCS-BOTTOM-UP (a, b)
  let l[0..m, 0..n] be an empty array
  m = len(a), n = len(b)

  for i = 0 to m
    l[i,0] = 0
  for j = 0 to n
    l[0,j] = 0

  for i = 1 to m
    for j = 1 to n
      if a[i] == b[j]
        l[i,j] = 1 + l[i-1,j-1]
      else
        l[i,j] = max(l[i-1, j], l[i, j-1])
  return l[i,j]
```

### 4. Construct an optimal solution from the computed information.

```
LCS-BOTTOM-UP-EXTENDED (a, b)
  let l[0..m, 0..n] be an empty array
  let c[1..m, 1..n] be an empty array
  m = len(a), n = len(b)

  for i = 0 to m
    l[i,0] = 0
  for j = 0 to n
    l[0,j] = 0

  for i = 1 to m
    for j = 1 to n
      if a[i] == b[j]
        l[i,j] = 1 + l[i-1,j-1]
        c[i,j] = "up-left"
      else if l[i-1, j] > l[i, j-1]
        c[i,j] = "up"
        l[i,j] = l[i-1, j]
      else
        c[i,j] = "left"
        l[i,j] = l[i, j-1]
  return l, c

LCS-PRINT (a, b)
  l, c = LCS-BOTTOM-UP-EXTENDED(a, b)
  m = len(a), n = len(b)
  while m > 0 and n > 0
    if c[m,n] == "up-left"
      defer print a[m]
    else if c[m,n] == "up"
      m -= 1
    else
      n -= 1
```

