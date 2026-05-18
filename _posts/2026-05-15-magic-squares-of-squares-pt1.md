---
layout: post
title: "Magic Square of Squares Part 1"
date: 2026-05-16
categories: math
---

The [Magic Square of Squares](https://en.wikipedia.org/wiki/Magic_square_of_squares) asks for a \\(3 \times 3\\) Magic Square where all the entries are square numbers.

To be a \\(3 \times 3\\) Magic Square \\(9\\) numbers are arranged in a \\(3 \times 3\\) square and each column, row, and diagonal add up to the same value. For instance, 

$$\begin{matrix}
2 & 7 & 6 \\
9 & 5 & 1 \\
4 & 3 & 8 \\
\end{matrix}$$

Is a magic square since each row, column, and diagonal sum to \\(15\\). However, in this case, not every element is a square number so it is not a "Magic Square of Squares". No \\(3 \times 3\\) "Magic Square of Squares" is known. I had made [a problem](https://conjecscore.org/problems/magicsos) related to finding one on [conjecscore.org](https://conjecscore.org/) but had not put really much thought into the problem.

I decided to spend a little time thinking about it, and decided my first goal would be to find potential values for the center entry.

I am going to sieve out various impossible chocies for the center entry. That is, a center entry is impossible if it could not possibly be used as a center entry in a "Magic Square of Squares". To do so, I will leverage two well known properties that *any* magic square must have:

1. Let the sum of any row, column, or digonal be called the magic sum and denote it as \\(S\\). Then the [center value of a magic square is precisely \\(S/3\\)](https://math.stackexchange.com/a/5034311/15140).
2. Furthermore, up to symmetry, [all magic squares must have the form:](https://math.stackexchange.com/a/5108848/15140)

$$\begin{matrix}
\frac S3 + a & \frac S3 - a - b & \frac S3 + b \\
\frac S3 - a + b & \frac S3 & \frac S3 + a - b \\
\frac S3 - b & \frac S3 + a + b & \frac S3 - a \\
\end{matrix}$$

Now, we may derive some equalities from the fact that all entries must be square. Since we are interested in the center square, denote it as \\(n^2\\). Furthermore, both \\(n^2 + a\\) and \\(n^2 - a\\) must be squares. Thus,

$$n^2 + a + n^2 - a = t^2 + f^2$$

on the other hand,

$$n^2 + a + n^2 - a = 2n^2$$

Combining the two equations shows that \\(n^2\\) is the average of two squares, that is:

$$n^2 = \frac{t^2 + f^2}{2}$$

Furthermore \\(n = \sqrt{\frac{t^2 + f^2}{2}} = \frac{\sqrt{t^2  + f^2}}{\sqrt{2}}\\). This might seem like a silly observation, but we may use the fact that \\(n\\) is an integer to observe that the prime factorization of \\(t^2 + f^2\\) must have an *odd* power of \\(2\\) in it.

There is a [quick way to compute the largest power of \\(2\\) that divides a number:](https://www.geeksforgeeks.org/dsa/highest-power-of-two-that-divides-a-given-number/)

```py
def largest_two_pow(n):
    return n & (~(n - 1))
```

This gives a nice sieve for possible center entry values:

- Range over \\(t\\) and \\(f\\).
- Compute \\(t^2 + f^2\\) and check if the largest dividing power of \\(2\\) is odd.
- If so, factor out one \\(2\\) and see if it is a perfect square.

When it comes to ranging over \\(t\\) and \\(f\\), this is just an arbitrary size range of numbers we want to check, however, since \\(t = n^2 + a > n^2 - a > f\\) we may impose the restriction \\(t > f\\) in the search. Thus, in code, we have something like:

```py
from math import isqrt


def largest_two_pow(n):
    return n & (~(n - 1))

def sqrt_int(x):
    isqr_x = isqrt(x)
    return isqr_x ** 2 == x

def main():
    RANGE = 20000
    candidate_n = set()
    for t in range(1, RANGE):
        for f in range(1, t):
            tf2 = t * t + f * f
            if largest_two_pow(tf2) % 2 == 1:
                tf2 //= 2
                if sqrt_int(tf2):
                    candidate_n.add(isqrt(tf2))
    print(len(candidate_n))
```

A Small Adaptation for a Better Sieve
-------------------------------------

The output indicates there are \\(2000\\) candidate values for \\(n\\) that work. In fact, we can adapt this program slightly to get an even smaller selection of \\(n\\).

We observed above that \\(n^2 + a\\) and \\(n^2 - a\\) were squares to conclude \\(n^2\\) was an average of a pair of different squares. It is also true that \\(n^2 + b\\) and \\(n^2 - b\\) are squares. Similarly, \\(n^2 + a + b\\) and \\(n^2 - a - b\\) are also squares. Thus, we can repeat this argument to see that \\(n^2\\) is the average of *at least* \\(3\\) different pairs of squares. So, we should only add to `candidate_n` if \\(n\\) appears \\(3\\) times or more. This can be easily done with a `Counter` object:

```py
from math import isqrt
from collections import Counter


def largest_two_pow(n):
    """Return the largest power of 2 that divides n."""
    return n & (~(n - 1))

def sqrt_int(x):
    """Checks if sqrt(x) is integral."""
    isqr_x = isqrt(x)
    return isqr_x ** 2 == x

def main():
    # 1st round
    RANGE = 20000
    candidate_n = Counter()
    for t in range(1, RANGE):
        for f in range(1, t):
            tf2 = t * t + f * f
            if largest_two_pow(tf2) % 2 == 1:
                tf2 //= 2
                if sqrt_int(tf2):
                    # tf2 now equals n^2 add n to candidates
                    candidate_n[isqrt(tf2)] += 1

    result = set({key for key, value in candidate_n.items() if value >= 3})
    print(len(result))


if __name__ == "__main__":
    main()
```

Now there are only \\(147\\) choices of \\(n\\) available.

Conclusion
----------

Still nothing significant, but hopefully, after some more thought, I can get some better candidates and at least get a [decent score on conjecscore](https://conjecscore.org/problems/magicsos). I will continue thinking about this problem when I have more time. Secondly, next time, I will do a review of the literature (I like to try the problem by myself for a bit first though!)
