# functional programming

## map

[map docs](https://docs.python.org/3/library/functions.html#map)

- apply *function* to every item of *iterable* creating a new iterable
  `new_iterable = map(function, iterable)`

```python
def square(number):
    return number ** 2

numbers = [1, 2, 3, 4, 5]

squared = map(square, numbers)

list(squared)
[1, 4, 9, 16, 25]

--

numbers = [-2, -1, 0, 1, 2]

abs_values = list(map(abs, numbers))
abs_values
[2, 1, 0, 1, 2]


list(map(float, numbers))
[-2.0, -1.0, 0.0, 1.0, 2.0]

words = ["Welcome", "to", "Real", "Python"]

list(map(len, words))
[7, 2, 4, 6]

>>> first_it = [1, 2, 3]
>>> second_it = [4, 5, 6, 7]

>>> list(map(pow, first_it, second_it))
[1, 32, 729]

>>> list(map(lambda x, y: x - y, [2, 4, 6], [1, 3, 5]))
[1, 1, 1]

>>> list(map(lambda x, y, z: x + y + z, [2, 4], [1, 3], [7, 8]))
[10, 15]

>>> string_it = ["processing", "strings", "with", "map"]
>>> list(map(str.capitalize, string_it))
['Processing', 'Strings', 'With', 'Map']

>>> with_spaces = ["processing ", "  strings", "with   ", " map   "]
>>> list(map(str.strip, with_spaces))
['processing', 'strings', 'with', 'map']

>>> with_dots = ["processing..", "...strings", "with....", "..map.."]
>>> list(map(lambda s: s.strip("."), with_dots))
['processing', 'strings', 'with', 'map']

>>> import math
>>> numbers = [1, 2, 3, 4, 5, 6, 7]
>>> list(map(math.factorial, numbers))
[1, 2, 6, 24, 120, 720, 5040]

```


## itertools

## itertools functions

One type of function it produces is **infinite iterators**

| function | description |
|---|:---|
| Count | **counts up** infinitely from a value|
| Cycle | *infinitely iterates through an iterable* (for instance a list or string)|
| Repeat| **repeats an object**, either infinitely or a specific number of times|
| Takewhile| takes items from iterable while predicate function remains True |
| Chain | **combines** iterables |
| Accumulate |returns a **running total** of values in an iterable|


### count

```python
from itertools import count

for i in count(3): # counts up starting from 3
	print(i)
	if i>= 11:
		break

""" 3  4  5  6  7  8  9  10  11 """

```

### accumulate

```python
from itertools import accumulate, takewhile


nums = list(accumulate(range(8)))
print(nums)  # [0,  1,   3,   6,  10,   15,   21, 28]
             # [0, 0+1, 1+2, 3+3, 6+4, 10+5, 15+6, 21+7]

```

### takewhile

```python
print(list(takewhile(lambda x: x<=6, nums)))  # [0, 1, 3, 6]

# takewhile stops as soon as predicate == FALSE!
nums = [2, 4, 6, 7, 9, 8]  # will stop returning at hitting value 7

print(list(takewhile(lambda x: x%2==0, nums))) 
# [2, 4, 6]
```

### combinatoric functions & permutations

```python

from itertools import product, permutations

letters = ("A", "B")

list(product(letters, range(2)))
# [('A', 0), ('A', 1), ('B', 0), ('B', 1)]

list(permutations(letters))
# [('A', 'B'), ('B', 'A')]


letters = ("A", "B", "C")
list(permutations(letters))
"""
[('A', 'B', 'C'), ('A', 'C', 'B'), ('B', 'A', 'C'),
 ('B', 'C', 'A'), ('C', 'A', 'B'), ('C', 'B', 'A')]
"""

a={1, 2}
len(list(product(range(3), a))) # 6
list(product(range(3), a))
# [(0, 1), (0, 2), (1, 1), (1, 2), (2, 1), (2, 2)]
```

## filter

- apply boolean function to an iterable to generate a new iterable

```python
import math

def is_positive(num):
    return num >= 0


def sanitized_sqrt(numbers):
    cleaned_iter = map(math.sqrt, filter(is_positive, numbers))
    return list(cleaned_iter)


sanitized_sqrt([25, 9, 81, -16, 0])
[5.0, 3.0, 9.0, 0.0]

```

## reduce

- apply reduction function to an iterable to produce single cumulative value

reduce() takes two required arguments:

1. function can be any Python callable that accepts two arguments and returns a value.
2. iterable can be any Python iterable.

```python
import functools
import operator
import os

files = os.listdir(os.path.expanduser("~"))

functools.reduce(operator.add, map(os.path.getsize, files))
4377381
```

## map vs comprehensions & gen expression

```python
# Generating a list with map
list(map(function, iterable))

# Generating a list with a list comprehension
[function(x) for x in iterable]

--

# Transformation function
def square(number):
    return number ** 2

numbers = [1, 2, 3, 4, 5, 6]

# Using map()
list(map(square, numbers))
[1, 4, 9, 16, 25, 36]

# Using a list comprehension
[square(x) for x in numbers]
[1, 4, 9, 16, 25, 36]

--

# Transformation function
def square(number):
    return number ** 2

numbers = [1, 2, 3, 4, 5, 6]

# Using map()
map_obj = map(square, numbers)
map_obj
<map object at 0x7f254d180a60>

>>> list(map_obj)
[1, 4, 9, 16, 25, 36]

# Using a generator expression
gen_exp = (square(x) for x in numbers)
gen_exp
<generator object <genexpr> at 0x7f254e056890>

list(gen_exp)
[1, 4, 9, 16, 25, 36]
```

## closures

- helps us to reduce the use of global variables
- if we need many functions, then go for class (OOP)
- if only 1 extra method, an elegant solution would be to use a closure rather than a class

```python
def by_factor(factor):
    def multiply(number):
        return factor * number
    return multiply  # no () !
```
```python
>>> double = by_factor(2)
>>> double(2)
4
>>> triple = by_factor(3)
>>> triple(3)
9
```

### invoke outside scope

```python
import logging 
logging.basicConfig(filename='example.log',
                    level=logging.INFO) 
 
def logger(func): 
    def log_func(*args): 
        logging.info( 
            'Running "{}" with arguments {}'.format(func.__name__,
                                                    args)) 
        print(func(*args)) 
    return log_func             
 
def add(x, y): 
    return x+y 
 
def sub(x, y): 
    return x-y 
```

```python
>>> add_logger = logger(add)
>>> sub_logger = logger(sub)
>>> add_logger(3,3)
6
>>> sub_logger(12,4)
8
```
```bash
cat example.log 
INFO:root:Running "add" with arguments (3, 3)
INFO:root:Running "sub" with arguments (12, 4)
```

### using lambdas

```python
def by_factor(factor):
	return lambda number: factor * number
```
```python
>>> quadruple = by_factor(4)
>>> quadruple(4)
16
```

## decorators

add logic to function without changing it

```python
import pprint
import time
from collections import namedtuple
import statistics as st


def timer(func):
    def _timer(*args,**kwargs):
        start = time.time()
        result = func(*args,**kwargs)
        end = time.time()
        print(f"execution time: {end - start}")
        return result
    return _timer  # call inner function


@timer
def describe(data):
    summary = namedtuple("summary", ["mean", "median", "mode"])
    return summary(
        st.mean(data),
        st.median(data),
        st.mode(data)
    )


if __name__ == '__main__':
    sample = [10, 2, 4, 7, 9, 3, 9, 8, 6, 7]
    pprint.pp(describe(sample))
```
```output
execution time: 0.0011296272277832031
summary(mean=6.5, median=7.0, mode=7)
```

## factory

allow user input to translate to objects being created
by coupling the string to the class

select key which has class, call that with extra parentheses to initialize
a class instance

```python
dictionary["user_input"](*args,**kwargs) 
```


```python
class Circle:
    def __init__(self, radius):
        self.radius = radius
    # Class implementation...

class Square:
    def __init__(self, side):
        self.side = side
    # Class implementation...

def shape_factory(shape_name, *args, **kwargs):
    shapes = {"circle": Circle, "square": Square}
    return shapes[shape_name](*args, **kwargs)
```
