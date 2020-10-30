# Kaprekar-6174

## Problem

Kaprekar's constant is 6174, named after the Indian mathematician who first discovered this property:

- Take any four-digit number, using at least two different digits (leading zeros are allowed).
- Arrange the digits in descending and then in ascending order to get two four-digit numbers, adding leading zeros if necessary.
- Subtract the smaller number from the bigger number.
- Go back to step 2 and repeat.

The above process, known as Kaprekar's routine, will always reach its fixed point, 6174, in at most 7 iterations.

Write a function which accepts a 4 digit number represented as a string, and returns the number of iterations to get to '6174'. If a number is passed in which is not solvable (where all digits are the same), return -1.

## Test Cases

```
solution('5432')
# 3

solution('0001')
# 5

solution('6174')
# 0

solution('5555')
# -1
```

## Solution

``` python
def solution(number):
    return solutionEx(number, 0)

def solutionEx(number, iteration):
    
    if (number == '6174'):
        return iteration
    
    s = list(number)
    s.sort(reverse = True)
    biggest = int(''.join(s))
    s.sort()
    smallest = int(''.join(s))
    diff = biggest - smallest
    if diff == 0:
        return -1
    elif diff == int(number):
        return iteration
    else:
        return solutionEx(str(diff).rjust(4,'0'), iteration + 1)
    
assert solution('5432') == 3
assert solution('0001') == 5
assert solution('6174') == 0
assert solution('5555') == -1
```