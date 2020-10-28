# Primes

## Problem

Create a function `solution(n)` which takes in a positive integer between 2 and 1,000,000 and returns the number of prime numbers less than or equal to `n`.

For example, given an input of 12, the prime numbers less than / equal to 12 total 5:

```
2,3,5,7,11
```

## Test Cases

```
solution(12)
# 5

solution(1000)
# 168

solution(1000000)
# 78,498
```

## Solution

``` python
import math

def solution(n):
    primes = [2]
    for i in range(3, n + 1, 1):
        isPrime = False
        sqrt = math.sqrt(i)
        for j in primes:
            if j > sqrt:
                # is prime
                isPrime = True
                break
            if i % j == 0:
                # not a prime
                break
        if isPrime:
            primes.append(i)
    
    return len(primes)

assert solution(12) == 5
assert solution(1000) == 168
assert solution(1000000) == 78498
```
