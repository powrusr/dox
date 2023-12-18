# functional programming

## Higher-order, pure & anon functions

- Higher-order functions take other functions as arguments, or return them as results
- Pure functions have no side effects, and return a value that depends only on their arguments

```python
def pure_function(x, y):
    temp = x + 2*y
    return temp / (2*x + y)

# pure
def func(x):
    y = x**2
    z = x + y
    return z

# impure
some_list = []
def impure(arg):
    some_list.append(arg)
```

Pure functions are:
- easier to reason about and test.
- more efficient. Once the function has been evaluated for an input, the result can be stored and referred to the next time the function of that input is needed, reducing the number of times the function is called. This is called memoization.
- easier to run in parallel. 
- complicate the otherwise simple task of I/O, more difficult to write

## lambda

```python
""" lambda land

assign functions to variable = anonymous

pass function as an argument to another function
They can only do things that require a single expression
"""
def some_func(f, arg)
    return f(arg)

some_func(lambda x: 2*x*x, 5)

a = (lambda x: x*x), (8) # 64

# Lambda functions can be assigned to variables, and used like normal functions
double = lambda x: x*2
print(double(7))
# however, usually better to define function with def instead ...

triple = lambda x: x * 3
add = lambda x, y: x + y  # using 2 params

print(add(triple(3), 4)) # 13
```


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

```python
def add_five(x):
	return x+5

nums = [11, 22, 33, 44, 55]
result = list(map(add_five, nums)) # map(function, iterable)
# [16, 27, 38, 49, 60]

# using lambda:
result = list(map(lambda x: x+5, nums)) # [16, 27, 38, 49, 60]
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

```python
nums = [11, 22, 33, 44, 55]
result = list(filter(lambda x: x%2==0, nums)) # [22, 44]
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

## generator

- yield statement is used to define a generator, replacing the return of a function to provide a result to its caller without destroying local variables

```python

def countdown():
	i=5
	while i > 0:
		yield i  # return value i & continue
		i -= 1


for i in countdown():
    print(i)  # 5 4 3 2 1

	
# they can be infinite!
def infinite_sevens():
    while True:
        yield 7


for i in infinite_sevens():
	print(i)
	
# Finite generators can be converted into lists by passing them as arguments to the list function

def numbers(x):
	for i in range(x):
		if i%2 == 0:
			yield i


print(list(numbers(11))) # [0, 2, 4, 6, 8, 10]
```

*note**
Using generators results in improved performance, which is the result of the lazy (on demand) generation of values, which translates to lower memory usage.
Furthermore, we do not need to wait until all the elements have been generated before we start to use them. 

## recursion

- Solve problems that can be broken up into sub-problems of the same type
- eg: 5! (5 factorial) is 5 * 4 * 3 * 2 * 1 (120)
- n! = n * (n-1)!
- BASE CASE: 1! = 1 -> can be calculated without performing any more factorial function calls

**note**
The base case acts as the exit condition of the recursion

```python

def factorial(x):
	# base case
	if x == 1:
		return 1
	else:
		return x * factorial(x-1)


print(factorial(5))  # 120


"""fibonacci"""
def fibo(n):
    if n <= 1:
        return n  # returns 0 & 1's
    else:
        return fibo(n-1) + fibo(n-2)


number = 6
for i in range(6):
    print(fibo(i))
		


def power(x, y):
    if y == 0:
        return 1
    else:
        return x * power(x, y-1)


print(power(2, 3))  # 8


"""Recursion can also be indirect. One function can call a second, which calls the first, which calls the second, and so on. This can occur with any number of functions"""


def is_even(x):
	if x == 0:
		return True
	else:
		return is_odd(x-1)


def is_odd(x):
	return not is_even(x)  # not! else will also return True when odd


is_even(9)  # False
is_even(12) # True
is_odd(17)  # True

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



