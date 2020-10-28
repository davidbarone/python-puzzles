# Partitions

## Problem

In number theory, a partition of a positive integer `n` is a way of writing `n` as a sum of positive integers. Two sums that differ only in the order of the summands are considered the same partition. For example, 4 can be partitioned in 5 distinct ways:

```
4
3 + 1
2 + 2
2 + 1 + 1
1 + 1 + 1 + 1
```

Write a function called `solution` that takes a single integer parameter between 1 and 50, and returns the number of distinct partitions of that integer

## Test Cases

```
solution(4)
# 5

solution(10)
# 42

solution(50)
# 204226
```

## Solution

``` python
def solution(n):
        """Returns the partitions for a positive integer, n"""
        return len(partitions(n, n))

def partitions(n, m):
    """
        Generates partitions for an integer n, with maximum summand, m.
    """
    results = []
    if n == 1:
        results.append([1])
        return results
    elif m == 1:
        results.append([1] * n)
        return results
    else:
        for i in range(m, 0, -1):
            if i == n:
                results.append([i])
            elif i < n:
                for result in partitions(n - i, i):
                    results.append([i] + result)
        return results
    
assert solution(4) == 5
assert solution(10) == 42
assert solution(50) == 204226
```