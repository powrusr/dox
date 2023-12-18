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

## functools

### reduce

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

### wraps

```python
import functools

# keep track # times func ran
def measure_fn(original_function):
    @functools.wraps(original_function)  # keep docstrings & function name
    def wrapper_function(*args, **kwargs):
        print(f"Function {original_function.__name__}
        return original_function(*args, **kwargs)  # execute
        # print(f"this code adds functionality & after {original_function.__name__}")
    return wrapper_function
```

### lru_cache

```python
import functools

@functools.lru_cache(maxsize=None)  # least recently used cache -> e.g. no function calls 4 same values
@measure_fn
def fibonnaci(n):
    """Return a number in the Fibonacci sequence."""
    # todo: im gonna forget this
    if n<2:
        return n
    return fibonnaci(n-1) + fibonnaci(n-2)
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



