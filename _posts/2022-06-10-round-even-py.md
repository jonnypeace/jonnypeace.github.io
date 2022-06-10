---
title: Rounding Even with Python
date: 2022-06-09 13:00:00 -500
categories: [python, maths]
tags: [python,maths,rounding]
---

## The Concept
  
Well, this is a funny one. I started this due to an ASTM method requiring a round even decimal value, and after writing a long python function, I discovered the decimal module has this ability built in.. 

## Short Example

So... i'll start with the short version, because if anyone is reading this, this is the one i would choose.

```python
from decimal import *


def round_val(x, r):
    # set precision to 56 significant figures (whatever is required),
    # using the decimal module to round half even
    getcontext().prec = 56
    factor = Decimal(1) / 10 ** r
    num1 = Decimal(x).quantize(factor, rounding=ROUND_HALF_EVEN)
    return num1


# forever loop while run = y for testing purposes.
# calls the round_val function and passes in the parameters x and r.
run = "y"
while run == "y":
    x = Decimal(input('Decimal number for rounding: '))
    r = int(input('Number of decimal points: '))
    result = round_val(x, r)
    print(result)
    run = input("Do you want to continue? Type 'y' if you do: ")
```
## Long Example

Brace yourself for the over-coded long version....

```python
#!/usr/bin/python3

import re
import decimal
from decimal import *


def round_val(x, r):
    # finding index position of decimal point
    point = str(x).index(".")

    # finding length of decimal number as a string
    length = len(str(x))

    # left of the decimal
    left_d = str(x)[:point]

    # right of the decimal
    right_d = str(x)[point + 1:]

    # concatenate left and right of the decimal (avoid floating math problems)
    l_and_r = (left_d + right_d)

    # looking for the last to characters for conditional statements.
    last_two = str(l_and_r)[-2:]

    # result of modulus into 5 needs to be zero for rounding even to occur.
    mod1 = float(l_and_r) % 5

    # checking conditions are met for rounding even
    if re.findall("^[02468]?5$", last_two) and mod1 == 0.0:
        ans = round((float(l_and_r) - 5) / 10 ** (length - (point + 1)), r)
        # this syntax works along the lines of "{:.2f}".format(val, decimal_point). .2f would be 2 decimal points
        # i.e. "{:.decimal_pointf}".format(val, decimal_point)
        # for some more info https://mkaz.blog/code/python-string-format-cookbook/
        ans = "{:.{}f}".format(ans, r)
        # print("Answer is (1st if): " + ans)
        return ans
    elif re.findall("^[13579]5$", last_two) and mod1 == 0.0:
        ans = (float(l_and_r) + 5) / 10 ** (length - (point + 1))
        ans = "{:.{}f}".format(ans, r)
        # print("Answer is (2nd if): " + ans)
        return ans
    else:
        ans = "{:.{}f}".format(x, r)
        # print("Answer is (3rd if): " + ans)
        return ans


# forever loop while run = y
# calls the round_val function and passes in the parameters x and r.
# while loop for testing purposes.
run = "y"
while run == "y":
    x = Decimal(input('Decimal number for rounding: '))
    r = int(input('Number of decimal points: '))
    calc = round_val(x, r)
    print("Answer is: " + str(calc))
    # run = input("Do you want to continue? Type 'y' if you do: ")
    run = "y"
```

The reason for the long version, for those who are unwise to floating point maths in programming languages, is to correct some issues with the base 2 maths. A better explanation can be found here

[https://docs.python.org/3/tutorial/floatingpoint.html](https://docs.python.org/3/tutorial/floatingpoint.html)