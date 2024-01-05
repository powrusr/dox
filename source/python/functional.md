# functional programming

## pure function

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

## higher order

- Higher-order functions take other functions as arguments, or return them as results

```python
import random

def unstable_function():
    """Simulates an unstable function that fails 50% of the time."""
    if random.randint(0, 1) == 0:
        print("Function failed!")
        raise ValueError("Transient error!")
    else:
        print("Function succeeded!")
        return "Success!"


def retry(func, max_retries=3):
    for _ in range(max_retries):
        try:
            return func()
        except ValueError:
            continue
        
    return "Failed after max retries"
```
```python
result = retry(unstable_function, 10)
```
```output
Function failed!
Function failed!
Function failed!
Function failed!
Function failed!
Function succeeded!
```
## lazy vs eager evaluation

```python
eager_squares = [x * x for x in range(5)]
print(eager_squares)
[0, 1, 4, 9, 16]

lazy_squares = (x * x for x in range(5))
print(lazy_squares)
<generator object <genexpr> at 0x106110c10>

next(lazy_squares)
0
next(lazy_squares)
1
```

Haskell -> all functions are lazy by default
Python -> functions could be lazy but we need to be more deliberate in their implementation


```python
eager_squares = [x * x for x in range(5)]
lazy_squares = (x * x for x in range(5))

def lazy_evens(n):
    for i in range(n):
        if i % 2 == 0:
            yield i

gen_func = lazy_evens(10)

next(gen_func), next(gen_func), next(gen_func)
(0, 2, 4)
```

Chaining Lazy Operations

```python

def lazy_squares(n):
    for i in n:
        yield i ** 2

squared_evens = lazy_squares(lazy_evens(5))

next(squared_evens), next(squared_evens), next(squared_evens)
(0, 4, 16)
```
## immutability

wrong:

```python
from typing import List

def increment_elements(data: List[int], value: int):
    for i in range(len(data)):
        data[i] += value
    return data

numbers = [1, 2, 3]
incremented_numbers = increment_elements(numbers, 3)
print(incremented_numbers)  # [4, 5, 6]
print(numbers) # [4, 5, 6] OOPS! 
[4, 5, 6]
[4, 5, 6]
```

good:

```python
from typing import List

def increment_elements(data: List[int], value: int):
    # for i in range(len(data)):
    #     data[i] += value
    # return data
    
    return [item + value for item in data]

numbers = [1, 2, 3]
incremented_numbers = increment_elements(numbers, 3)
print(incremented_numbers)  # [4, 5, 6]
print(numbers)
[4, 5, 6]
[1, 2, 3]
```

- list comprehensions always create a new object, thus ensuring immutability

undo/redo operations with immutable data

```python
history = create_history() # create a new history object
history = edit(history, "Hello") # add a new edit to the history
history = edit(history, "Hello, World") # add another edit to the history
history = undo(history) # undo the last edit resulting history: "Hello"
history = redo(history) # redo the last edit resulting history: "Hello, World"
history = undo(history) # undo the last edit resulting history: "Hello"

def create_history():
    # states = ["", "Hello", "Hello, World"]
    # current_index = 1    
    
    return ([""], 0)

def edit(history, new_text):
    states, current_index = history
    new_states = states[:current_index + 1] + [new_text]
    return (new_states, current_index + 1)  

def undo(history):
    states, current_index = history
    new_index = max(0, current_index - 1)
    return (states, new_index)

def redo(history):
    states, current_index = history
    new_index = min(len(states) - 1, current_index + 1)
    return (states, new_index)

def current_state(history):
    states, current_index = history    
    return states[current_index]

history = create_history()

history = edit(history, "Duke")

print(current_state(history))
Duke


history = edit(history, "Duke Nukem")

print(current_state(history))
Duke Nukem


history = undo(history)

print(current_state(history))
Duke


history = redo(history)

print(current_state(history))
Duke Nukem


history = undo(history)
history = undo(history)

print(current_state(history))
```

## lambda

```python
lambda num: num * num
<function __main__.<lambda>(num)>

# how about a list of nums?

def square(num_list):
    return [num*num for num in num_list]

lambda num_list: [num*num for num in num_list]
<function __main__.<lambda>(num_list)>

# if even, then square, if odd then cube? get me the sum of all these.
def square(num_list):
    return sum([n**2 if n % 2 == 0 else n**3 for n in num_list])

lambda num_list: sum([n**2 if n % 2 == 0 else n**3 for n in num_list])
<function __main__.<lambda>(num_list)>
```

good & bad use

```python
def sum_squares_or_cubes(num_list):
    return sum([n**2 if n % 2 == 0 else n**3 for n in num_list])

def complex_operation(num_list):
    result = 1  # Initialize product result
    for num in num_list:
        if num % 2 == 0:
            result *= num**2  # Square if even
        else:
            result *= num**3  # Cube if odd
    return result
```
```python
from functools import reduce

complex_opeartion_lambda = lambda nl: reduce(lambda acc, n: acc * (n**2 if n%2==0 else n**3), nl, 1)

complex_opeartion([1,2,3,4,5])
216000

complex_opeartion_lambda([1,2,3,4,5])
216000
```
Nesting And In-Place Lambdas

```python
lambda s: s.capitalize()
<function __main__.<lambda>(s)>

(lambda s: s.capitalize())("hey there")
'Hey there'

(lambda x: "even" if x%2==0 else "odd")(4)
'even'

(lambda x: (lambda x: "even" if x%2==0 else "odd")(x).capitalize())(4)
'Even'
```
challenge

- ternary(5)  # Output: 'positive'
- ternary(-5)  # Output: 'negative'
- ternary(0)  # Output: 'zero'

```python
# a if condition (is true) else ( insert other ternary here ) 

ternary = lambda x: 'positive' if x > 0 else 'negative' if x < 0 else 'zero'

ternary(5)  # Output: 'positive'
'positive'

ternary(-5)  # Output: 'negative'
'negative'

ternary(0)  # Output: 'zero'
'zero'
```


## map

Transform Iterables

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

```python
numbers = range(1, 6)

def cube(x):
    return x**3

cubed = map(cube, numbers)

list(cubed)
[1, 8, 27, 64, 125]

cubed
<map at 0x1044c0be0>

cubed = map(lambda x: x**3, numbers)

list(cubed)
[1, 8, 27, 64, 125]

words = ('alice', 'hello', 'how', 'are', 'you')
capitalized = map(lambda x: x.capitalize(), words)

tuple(capitalized)
('Alice', 'Hello', 'How', 'Are', 'You')

floats = {1.2, 2.3, 3.4}

rounded = map(lambda x: round(x), floats)

set(rounded)
{1, 2, 3}
```
Mapping Over Multiple Iterables

```python
first_names = ['alice', 'bob', 'charlie']
last_names = ['smith', 'jones', 'brown']

full_names = list(map(lambda x, y: x.capitalize() + ' ' + y.capitalize(), first_names, last_names))

full_names
['Alice Smith', 'Bob Jones', 'Charlie Brown']

middle_names = ['jane', 'john', 'james']

full_names = list(map(lambda x, y, z: x.capitalize() + ' ' + y.capitalize() + ' ' + z.capitalize(), first_names, middle_names, last_names))

full_names
['Alice Jane Smith', 'Bob John Jones', 'Charlie James Brown']

full_names = list(map(lambda x, y, z: (x+' '+y+' '+z).title(), first_names, middle_names, last_names))

full_names
['Alice Jane Smith', 'Bob John Jones', 'Charlie James Brown']
Built-Ins

word = "hamming"

len(word)
7

words = ["hamming", "von neumann", "turing", "knuth"]

len(words)
4

list(map(len, words))
[7, 11, 6, 5]

num_strings = ["1", "2", "3", "4", "5"]

# list(map(str, range(1, 6)))
['1', '2', '3', '4', '5']

# cast to ints

type(int("2"))
int

list(map(int, num_strings))
[1, 2, 3, 4, 5]

# add 2 nums?

from operator import add, mul

a = range(1,4)
b = range(4,7)

list(map(add, a, b))
[5, 7, 9]

list(map(mul, a, b))
[4, 10, 18]
```

do this transformation:

```python
data = {
    'abc': 5,
    'def': 10,
    'ghi': 15
}
Output:

{
    'cba': 15,
    'fed': 20,
    'ihg': 25
}
```
```python
data = {
    'abc': 5,
    'def': 10,
    'ghi': 15
}

dict(map(lambda kv: (kv[0][::-1], kv[1]+10), data.items()))
{'cba': 15, 'fed': 20, 'ihg': 25}
```
## zip

zip creates a generator with tuples

```python
cars = ['Ford', 'Chevy', 'Toyota', 'Tesla']
prices = [20000, 25000, 30000, 50000]

zipped_data = zip(cars, prices)

type(zipped_data)
zip

next(zipped_data)
('Chevy', 25000)

list(zipped_data)
[('Ford', 20000), ('Chevy', 25000), ('Toyota', 30000), ('Tesla', 50000)]

colors = ['red', 'blue', 'black', 'white']

zipped_data = zip(cars, prices, colors)

list(zipped_data)
[('Ford', 20000, 'red'),
 ('Chevy', 25000, 'blue'),
 ('Toyota', 30000, 'black'),
 ('Tesla', 50000, 'white')]
```

### strict mode

```python
cars = ['Ford', 'Chevy', 'Toyota', 'Tesla']
prices = [20000, 25000, 30000, 50000]
colors = ['red', 'blue', 'black', 'white']
zipped_data = zip(cars, prices, colors)

list(zipped_data)
[('Ford', 20000, 'red'),
 ('Chevy', 25000, 'blue'),
 ('Toyota', 30000, 'black'),
 ('Tesla', 50000, 'white')]

cars = ['Ford', 'Chevy', 'Toyota']
prices = [20000, 25000, 30000, 50000]
colors = ['red', 'blue', 'black', 'white']

zipped_data = zip(cars, prices, colors)
list(zipped_data)
[('Ford', 20000, 'red'), ('Chevy', 25000, 'blue'), ('Toyota', 30000, 'black')]

zipped_data = zip(cars, prices, colors, strict=True)

list(zipped_data)
ValueError: zip() argument 2 is longer than argument 1
```
### using *

```python
zipped_data = [('Alice', 85), ('Bob', 90), ('Charlie', 78)]

names = []
scores = []

for name, score in zipped_data:
    names.append(name)
    scores.append(score)

names, scores
(['Alice', 'Bob', 'Charlie'], [85, 90, 78])

names, scores = zip(*zipped_data)

names, scores
(('Alice', 'Bob', 'Charlie'), (85, 90, 78))

[('Alice', 85), ('Bob', 90), ('Charlie', 78)]
[('Alice', 85), ('Bob', 90), ('Charlie', 78)]
# * -> ('Alice', 85), ('Bob', 90), ('Charlie', 78)

# zip(iter1, iter2, iter3)

list1, list2 = zip(('Alice', 85), ('Bob', 90), ('Charlie', 78))

list1
('Alice', 'Bob', 'Charlie')

list2
(85, 90, 78)
```
### build dict

```python
stock_index = ["s_and_p", "dow_jones", "nasdaq"]
close_values = [2470.30, 22000.00, 6312.47]

index_dict = {}

for index, close_value in zip(stock_index, close_values):
    index_dict[index] = close_value

index_dict
{'s_and_p': 2470.3, 'dow_jones': 22000.0, 'nasdaq': 6312.47}

{idx: val for idx, val in zip(stock_index, close_values)}
{'s_and_p': 2470.3, 'dow_jones': 22000.0, 'nasdaq': 6312.47}

dict(zip(stock_index, close_values))
{'s_and_p': 2470.3, 'dow_jones': 22000.0, 'nasdaq': 6312.47}
```

### functional pipelining

```python
data = [1, 2, 3, 4, 5]

# operation 1: double
# operation 2: square root, rounded to 2 decimals

map(lambda x: x * 2, data)
<map at 0x110a7a6e0>

map(lambda x: round(x**0.5, 2), data)
<map at 0x110a783d0>

pipeline = zip(
    map(lambda x: x * 2, data),
    map(lambda x: round(x**0.5, 2), data)
)

list(pipeline)
[(2, 1.0), (4, 1.41), (6, 1.73), (8, 2.0), (10, 2.24)]

pipeline = (
    (
        (lambda x: x * 2)(x),
        (lambda x: round(x**0.5, 2))(x)
    )
    for x in data
)

# next()

list(pipeline)
[(2, 1.0), (4, 1.41), (6, 1.73), (8, 2.0), (10, 2.24)]

def double(x):
    return x * 2

def square_root(x):
    return round(x**0.5, 2)

pipeline = (
    (
        double(x),
        square_root(x)
    )
    for x in data
)

list(pipeline)
[(2, 1.0), (4, 1.41), (6, 1.73), (8, 2.0), (10, 2.24)]
```
### challenge

instructions

```
# use strict mode
Input Data
items = ['Shirt', 'Pants', 'Shoes', 'Hat']
prices = [20.00, 30.00, 50.00, 10.00]
colors = ['Red', 'Blue', None, 'Black']
discounts = [10, 0, 20, 5]
Target Output
{
    'Shirt': {
        'Price': 20.00,
        'Available Colors': 'Red',
        'Discounted Price': 18.00
    },
    'Pants': {
        'Price': 30.00,
        'Available Colors': 'Blue',
        'Discounted Price': 30.00
    },
    'Shoes': {
        'Price': 50.00,
        'Available Colors': None,
        'Discounted Price': 40.00
    },
    'Hat': {
        'Price': 10.00,
        'Available Colors': 'Black',
        'Discounted Price': 9.50
    }
}
```
```python
items = ['Shirt', 'Pants', 'Shoes', 'Hat']
prices = [20.00, 30.00, 50.00, 10.00]
colors = ['Red', 'Blue', None, 'Black']
discounts = [10, 0, 20, 5]

# if not(len(items) == len(prices) == len(colors)...)
    # raise ValueError


try:
    product_tuples = list(zip(items, prices, colors, discounts, strict=True))
except ValueError:
    raise ValueError("All input lists must have the same length!")

product_tuples
[('Shirt', 20.0, 'Red', 10),
 ('Pants', 30.0, 'Blue', 0),
 ('Shoes', 50.0, None, 20),
 ('Hat', 10.0, 'Black', 5)]

report = {
    item: {
        'Price': price,
        'Available Colors': color,
        'Discounted Price': round(price * (1-discount/100), 2)
    } for item, price, color, discount in product_tuples
}

report
{'Shirt': {'Price': 20.0, 'Available Colors': 'Red', 'Discounted Price': 18.0},
 'Pants': {'Price': 30.0,
  'Available Colors': 'Blue',
  'Discounted Price': 30.0},
 'Shoes': {'Price': 50.0, 'Available Colors': None, 'Discounted Price': 40.0},
 'Hat': {'Price': 10.0, 'Available Colors': 'Black', 'Discounted Price': 9.5}}
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

```python
words = ["apple", "banana", "cherry", "date", "eggplant", "fig", "grape"]

short_words = filter(lambda w: len(w) <= 5, words)

list(short_words)
['apple', 'date', 'fig', 'grape']
```
### multiple conditions

```python
# even numbers between 1 and 10, inclusive

numbers = range(1, 11)

filtered_numbers = filter(lambda x: x % 2 == 0, numbers)
list(filtered_numbers)
[2, 4, 6, 8, 10]

# even numbers between 1 and 10, inclusive... that are also greater than 5

filtered_numbers = filter(lambda x: x % 2 == 0 and x > 5, numbers)
list(filtered_numbers)
[6, 8, 10]

books = [
    {"title": "The Hobbit", "genre": "Fantasy", "rating": 4.7},
    {"title": "Dune", "genre": "Sci-Fi", "rating": 4.5},
    {"title": "To Kill a Mockingbird", "genre": "Classics", "rating": 4.9},
    {"title": "Eragon", "genre": "Fantasy", "rating": 4.1},
    {"title": "The Catcher in the Rye", "genre": "Classics", "rating": 3.8},
    {"title": "Mistborn", "genre": "Fantasy", "rating": 4.4}
]

# selection criteria:
# - genre must be fantasy
# - rating min 4.5
# - title must come before letter M

def is_genre(book, genre):
    return book["genre"] == genre

def has_min_rating(book, min_rating):
    return book["rating"] >= min_rating

def is_alpha_before(book, letter):
    return book["title"][0].lower() < letter.lower()

list(
    filter(
    lambda book:
        is_genre(book, "Fantasy") and
        has_min_rating(book, 4.5) and
        is_alpha_before(book, "W")
    , books)
)
[{'title': 'The Hobbit', 'genre': 'Fantasy', 'rating': 4.7}]
```

### with nested lambdas

```python
books = [
    {"title": "The Hobbit", "genre": "Fantasy", "rating": 4.7},
    {"title": "Dune", "genre": "Sci-Fi", "rating": 4.5},
    {"title": "To Kill a Mockingbird", "genre": "Classics", "rating": 4.9},
    {"title": "Eragon", "genre": "Fantasy", "rating": 4.1},
    {"title": "The Catcher in the Rye", "genre": "Classics", "rating": 3.8},
    {"title": "Mistborn", "genre": "Fantasy", "rating": 4.4}
]

# outer lambda (book)
#     inner lambda 1 (book, arg1)
#     inner lambda 2 (book, arg2)
#     inner lambda 3 (book, arg3)

list(
    filter(
    lambda book:
        (lambda book, genre: book["genre"] == genre)(book, "Fantasy") and
        (lambda book, rating: book["rating"] >= rating)(book, 4.5) and
        (lambda book, letter: book["title"][0].lower() < letter.lower())(book, "W")
    , books
    )
)
[{'title': 'The Hobbit', 'genre': 'Fantasy', 'rating': 4.7}]
list(
    filter(
    lambda book:
        is_genre(book, "Fantasy") and
        has_min_rating(book, 4.5) and
        is_alpha_before(book, "W")
    , books)
)
```
### chained filtering

```python
books = [
    {"title": "The Hobbit", "genre": "Fantasy", "rating": 4.7},
    {"title": "Dune", "genre": "Sci-Fi", "rating": 4.5},
    {"title": "To Kill a Mockingbird", "genre": "Classics", "rating": 4.9},
    {"title": "Eragon", "genre": "Fantasy", "rating": 4.1},
    {"title": "The Catcher in the Rye", "genre": "Classics", "rating": 3.8},
    {"title": "Mistborn", "genre": "Fantasy", "rating": 4.4}
]

# Selection criteria:
#  - the book must be fantasy
#  - it must be rated at least 4.5 
#  - the title must be alphabetically before W

fantasy_books = filter(lambda book: book["genre"] == "Fantasy", books)
highly_rated = filter(lambda book: book["rating"] >= 4.5, books)
alpha_filtered = filter(lambda book: book["title"] < "W", books)

# chaining/pipeing

fantasy_books = filter(lambda book: book["genre"] == "Fantasy", books)
highly_rated = filter(lambda book: book["rating"] >= 4.5, fantasy_books)
alpha_filtered = filter(lambda book: book["title"] < "W", highly_rated)

list(alpha_filtered)
[{'title': 'The Hobbit', 'genre': 'Fantasy', 'rating': 4.7}]
```

### exercise

```python
reference = [0, 1, 1, 0, 1, 0]
values = ["a", "b", "c", "d", "e", "f"]

# The function should return: ['b', 'c', 'e']
```
```python
reference = [0, 1, 1, 0, 1, 0]
values = ["a", "b", "c", "d", "e", "f"]
#  zip(reference, values) -> (0, "a"), (1, "b"),...

list(
    filter(lambda rv: rv[0] == 1, zip(reference, values))
)
[(1, 'b'), (1, 'c'), (1, 'e')]

list(v for _, v in filter(lambda rv: rv[0] == 1, zip(reference, values)))
['b', 'c', 'e']
```
alternative

```python
reference = [0, 1, 1, 0, 1, 0]
values = ["a", "b", "c", "d", "e", "f"]

list(v for _, v in filter(lambda rv: rv[0], zip(reference, values)))
['b', 'c', 'e']

from collections import namedtuple

Pair = namedtuple("Pair", ["select", "value"])

pair = Pair(0, 'a')

pair.select
0

bool(pair.select)
False

pair.value
'a'

reference = [0, 1, 1, 0, 1, 0]
values = ["a", "b", "c", "d", "e", "f"]

list(map(Pair, map(bool, reference), values))
[Pair(select=False, value='a'),
 Pair(select=True, value='b'),
 Pair(select=True, value='c'),
 Pair(select=False, value='d'),
 Pair(select=True, value='e'),
 Pair(select=False, value='f')]

pairs = map(Pair, map(bool, reference), values)

[p.value for p in filter(lambda p: p.select, pairs)]
['b', 'c', 'e']
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

## any all

### filter and map
```python
ages = [32, 19, 24, 53, 17]

# is there a single person who could order a drink?
# assume drinking age is 21

any(map(lambda age: age >= 21, ages))
True

all(map(lambda age: age >= 21, ages))
False

books = [
    {"title": "Book A", "pages": 150, "price": 15},
    {"title": "Book B", "pages": 120, "price": 12},
    {"title": "Book C", "pages": 105, "price": 20},
    {"title": "Book D", "pages": 250, "price": 25},
]

filtered_books = filter(lambda book: book["pages"] > 100 and book["price"] > 10, books)

all(book in filtered_books for book in books)
True

all(map(lambda book: book["pages"] > 100 and book["price"] > 10, books))
True
```
### negation

```python
names = ["Alice", "Andy", "Bob", "Ann", "Charlie"]

all(name.startswith("A") for name in names)
False

not all(name.startswith("A") for name in names)
True

any(not name.startswith("A") for name in names)
True

strings = ["Hello", " ", "World", "", "\t", "Python"]

# str.strip() -> " " -> "" -> bool("") -> False -> any(False, False...) -> False

any(not s.strip() for s in strings)
True
```
### short-circuiting

```python
nums = [1, 10, 1, 4, 2, 2, 1, 12, 4, 9]

any(num > 5 for num in nums)
True

def slow_check(x):
    print(f"checking {x}")
    return x > 5

any(slow_check(n) for n in nums)
checking 1
checking 10

True
```
## comprehensions

```python
"""
build this matrix

0 1 2
0 1 2
0 1 2
"""

matrix = [[j for j in range(3)] for i in range(3)]

matrix
[[0, 1, 2], [0, 1, 2], [0, 1, 2]]

[item for row in matrix for item in row]
[0, 1, 2, 0, 1, 2, 0, 1, 2]

flat_matrix = []

for row in matrix:
    for item in row:
        flat_matrix.append(item)

flat_matrix
[0, 1, 2, 0, 1, 2, 0, 1, 2]
```

### multiple iterables

```python
list1 = [1, 2, 3]
list2 = [3, 2, 1]

# target output: [
#     [1, 3], [1, 2], [1, 2], [2, 3],...
# ]

[ [x, y] for x in list1 for y in list2 ]
[[1, 3], [1, 2], [1, 1], [2, 3], [2, 2], [2, 1], [3, 3], [3, 2], [3, 1]]

[ [x, y] for x in list1 for y in list2 if x < y ]
[[1, 3], [1, 2], [2, 3]]
```

### exercise

Transform this list of customers dictionaries into a list of customer names with active accounts who have spent more than 100.
```python
customers = [
    {"name": "Alice", "account_status": "active", "amount_spent": 150},
    {"name": "Bob", "account_status": "inactive", "amount_spent": 200},
    {"name": "Charlie", "account_status": "active", "amount_spent": 50},
    {"name": "Diana", "account_status": "active", "amount_spent": 300}
]

# define the eligibility criteria as a func
def is_eligible(customer):
    return customer["account_status"] == "active" and customer["amount_spent"] >100

# filter the list using that func
eligible_customers = filter(is_eligible, customers)

# create the new list from the filtered gen
eligible_names = [customer["name"] for customer in eligible_customers]
eligible_names
['Alice', 'Diana']

[c
['Alice', 'Diana']
```
## set comprehensions

```python
# sets vs lists?
    # sets: UNORDERED collections of UNIQUE elements

set1 = {1, 2, 3, 4, 5}
list1 = [1, 2, 3, 4, 5]

3 in set1
True

3 in list1
True

set1 = {1, 2, 3, 4, 5}
set2 = {3, 4, 5, 6, 7}

set1.difference(set2)
{1, 2}

set1.intersection(set2)
{3, 4, 5}

numbers = [1, 2, 2, 3, 4, 4, 4, 5, 5, 5]

{ n**2 for n in numbers }
{1, 4, 9, 16, 25}

{ n**2 for n in set(numbers) }
{1, 4, 9, 16, 25}

{ n**2 for n in set(numbers) if n % 2 == 0 }
{4, 16}

# More Advanced Operations

set1 = {1, 2, 3, 4, 5}
set2 = {4, 5, 6, 7, 8}

set1.intersection(set2)
{4, 5}

set1 & set2
{4, 5}

{n for n in set1 if n in set2}
{4, 5}

words = ['apple', 'banana', 'cherry', 'orange']

# vowels -> aeiou

{letter for word in words for letter in word if letter in 'aeiou'}
{'a', 'e', 'o'}
```
### punctuation

```python
sentence = "Hello, World! This is a test. Isn't it great to explore Python's capabilities? Yes, indeed!"

from string import punctuation

print(punctuation)
"""!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~"""


# str.strip

"Andy".strip("y")
'And'

"Wor;ld!;".strip("!,:;")
'Wor;ld'

"hello there".split()
['hello', 'there']

{len(word.strip(punctuation)) for word in sentence.split()}
{1, 2, 3, 4, 5, 6, 7, 8, 12}
```
### primes

```python
n = 40

# find all the primes less than n

{ x for x in range(2, n) if not any(x % y == 0 for y in range(2, int(x**0.5) + 1)) }
{2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37}
```

## dictionary comprehensions

```python
# { key_expression : value_expression for item in iterable }

words = ['apple', 'banana', 'cherry'] 

words[0][::-1]
'elppa'

# { 'apple': 'elppa', ...}

{w:w[::-1] for w in words}
{'apple': 'elppa', 'banana': 'ananab', 'cherry': 'yrrehc'}

{w:w[::-1] for w in words if len(w) > 5}
{'banana': 'ananab', 'cherry': 'yrrehc'}

{ w : w[::-1] if len(w) > 5 else w for w in words}
{'apple': 'apple', 'banana': 'ananab', 'cherry': 'yrrehc'}
More Use Cases

original = {'a': 1, 'b': 2, 'c': 3}

{
    k: v+1 for k, v in original.items()
}
{'a': 2, 'b': 3, 'c': 4}

nested = {'first': {'a': 1, 'b': 2}, 'second': {'c': 3, 'd': 4}}

flattened = {'first_a': 1, 'first_b': 2, 'second_c': 3, 'second_d': 4}

{
    f"{outer_key}_{inner_key}": inner_val
    
    for outer_key, inner_dict in nested.items() 
        for inner_key, inner_val in inner_dict.items()
}
{'first_a': 1, 'first_b': 2, 'second_c': 3, 'second_d': 4}

d = {'a': 1, 'b': 2, 'c': 3}

{ v:k for k, v in d.items() }
{1: 'a', 2: 'b', 3: 'c'}
```
### exercise

```console
Example Input:

d1 = {'a': 1, 'b': 2}
d2 = {'b': 3, 'c': 4}
Expected Output:

{'a': 1, 'b': 5, 'c': 4}  # 'b' is the sum of 2 (from d1) and 3 (from d2); 'a' and 'c' are unique to d1 and d2, respectively.
```
```python
d1 = {'a': 1, 'b': 2}
d2 = {'b': 3, 'c': 4}

# ...we want the output to be:

merged = {'a': 1, 'b': 5, 'c': 4}
[17]
d1.keys()
dict_keys(['a', 'b'])
[18]
d2.keys()
dict_keys(['b', 'c'])
[19]
d1.keys() | d2.keys()
{'a', 'b', 'c'}
# where k shows up in both: sum
# where k shows up in left: left value
# where k shows up in right: right value
[20]
{
    k: d1[k] + d2[k] if (k in d1 and k in d2)
    else d1[k] if k in d1
    else d2[k]        
    for k in d1.keys() | d2.keys()
}
{'c': 4, 'a': 1, 'b': 5}
# merged = {'a': 1, 'b': 5, 'c': 4}
```
or
```python
d1 = {'a': 1, 'b': 2}
d2 = {'b': 3, 'c': 4}

# we want the output to be:

merged = {'a': 1, 'b': 5, 'c': 4}

# .get()

d1['a']
1

d1.get('aaslfkjas', 2)
2

d1['aaslfkjas']
KeyError: 'aaslfkjas'

{
    k: d1.get(k, 0) + d2.get(k, 0)     
    for k in d1.keys() | d2.keys()
}
{'c': 4, 'a': 1, 'b': 5}
# where k shows up in both: sum
# where k shows up in left: left value
# where k shows up in right: right value

d1
{'a': 1, 'b': 2}

set(d1)
{'a', 'b'}

{
    k: d1.get(k, 0) + d2.get(k, 0)
    for k in set(d1) | set(d2)
}
{'c': 4, 'a': 1, 'b': 5}
```
## iterator

iterator vs iterable:
* iterable: an object that could be iterated upon, a collection which could be looped over
* iterator: the mechanism that performs the iteration
* every iterator is an iterable, but not vice versa

```python
my_list = [1, 2, 3]

next(my_list)
TypeError: 'list' object is not an iterator

my_iter = iter(my_list)

my_iter
<list_iterator at 0x10a289420>

next(my_iter)
1

next(my_iter)
2

next(my_iter)
3

next(my_iter)
StopIteration:

my_tuple = ('apple', 'banana', 'cherry')

tuple_iter = iter(my_tuple)

while True:
    try:
        fruit = next(tuple_iter)
        print(fruit)
    except StopIteration:
        break
apple
banana
cherry
```
### iterator protocol

what is a the iterator protocol?
* __iter__() - initializes the iteration
* __next__() - returns the next iterm from the collection; raises StopIteration when exhausted

```python
# FunctionalIterator(start_value, end_value, func) -> 
squares_iterator = FunctionalIterator(1, 6, square)

for num in suqares_iterator:
    print(num)
    
# 1, 4, 9, 16, 25

class FunctionalIterator:
    def __init__(self, start, end, function):
        self.current = start
        self.end = end
        self.function = function
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current < self.end:
            result = self.function(self.current)
            self.current += 1
            return result
        else:
            raise StopIteration


square = lambda x: x**2

squares_iterator = FunctionalIterator(1, 6, square)

for num in squares_iterator:
    print(num)
    
# 1, 4, 9, 16, 25
1
4
9
16
25
```

## generator

* an iterable, but of a unique kind
* indexing is NOT allowed
* iteration with for loops or next is 

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

```python
def even_numbers(n):
    for i in range(n):
        if i % 2 == 0:
            yield i

for number in even_numbers(10):
    print(number) # 0, 2, 4, 6, 8
0
2
4
6
8

gen = even_numbers(10)

next(gen)
0

next(gen)
2
```
### generator expressions

* generator functions: like regular functions but with yield instead of return
* generator expressions: like list/set comprehensions but with () instead of []

```python
# even numbers up to a limit

gen_expr = (i for i in range(10) if i % 2 == 0)

next(gen_expr)
2

for number in gen_expr:
    print(number)
0
2
4
6
8


sum(i for i in range(10) if i % 2 == 0)
20
```

### 2-way communication

generators can also consume data!
* implemented via .send()
* needs to be primed first
* 2-way generators are typically called coroutines ("coros")
* exceedingly popular in async programming 
* in functional programming, the 1-way paradigm of generators as producers of data reigns

```python
gen = echo()
next(gen)
gen.send("hello there") # "you said: hello there"


def echo():
    while True:
        received = yield
        print("you said: ", received)


# truthy counter generator 
def counter():
    count = 0
    while True:
        received = yield count
        if received:
            count += 1

count_gen = counter()
next(count_gen) # priming
0

count_gen.send("hey")
1
count_gen.send(None)
1
count_gen.send("Item1")
2
count_gen.send(False)
2
count_gen.send("Another item")
3
```

```python
def fib():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b
        
        # equivalent to:
        # temp = a
        # a = b
        # b = temp + b
        
gen = fib()
for _ in range(20):
    print(next(gen))
0
1
1
2
3
5
8
13
21
34
55
89
144
233
377
610
987
1597
2584
4181
```
### exercise

Sliding Window Fibonacci With Deque

```python
def fib():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

import collections
collections.deque

from collections import deque

window = deque(maxlen=3)

gen = fib()

# last n fibs less than or equal to some m, e.g last 3 fib <= 20

def fib_sliding_window(n, window_size=3):
    gen = fib()
    window = deque(maxlen=window_size)
    
    for number in gen:
        window.append(number)
        if len(window) == window_size:
            print(window)
            if window[-1] >= n:
                break

fib_sliding_window(30)
deque([0, 1, 1], maxlen=3)
deque([1, 1, 2], maxlen=3)
deque([1, 2, 3], maxlen=3)
deque([2, 3, 5], maxlen=3)
deque([3, 5, 8], maxlen=3)
deque([5, 8, 13], maxlen=3)
deque([8, 13, 21], maxlen=3)
deque([13, 21, 34], maxlen=3)
```

refactor: what if we want to start from an arbitrary set of numbers, rather than alway 0, 1?

```python
def fib(start_points=(0,1)):
    a, b = start_points
    while True:
        yield a
        a, b = b, a + b

def fib_sliding_window(n, window_size=3, start_points=(0,1)):
    gen = fib(start_points)
    window = deque(maxlen=window_size)
    
    for number in gen:
        window.append(number)
        if len(window) == window_size:
            print(window)
            if window[-1] >= n:
                break

fib_sliding_window(30, start_points=(6, 9))
deque([6, 9, 15], maxlen=3)
deque([9, 15, 24], maxlen=3)
deque([15, 24, 39], maxlen=3)
```

### data pipelining

```console
logfile.log

2024-06-01 09:15:34 INFO Server started successfully
2024-06-01 09:17:42 ERROR Unable to connect to database
2024-06-02 09:18:05 INFO Received request from 192.168.1.10
2024-06-02 09:19:20 ERROR Disk space low on drive C:
2024-06-03 09:21:00 INFO User admin logged in
2024-06-03 09:22:15 WARNING CPU usage is high
2024-06-04 09:23:30 ERROR Failed to send email notification
2024-06-04 09:25:45 INFO Scheduled maintenance starts in 10 minutes
2024-06-05 09:27:00 ERROR Backup process failed
2024-06-05 09:30:20 INFO New connection from 192.168.1.15
```
```python
# make sure it's in your working dir
import os
list(filter(lambda f: '.log' in f, os.listdir()))
['logfile.log']

def read_lines(filename):
    with open(filename, 'r') as f:
        for line in f:
            yield line.strip()

# line -> yes/no filter

def filter_errors(lines):
    for line in lines:
        if "ERROR" in line:
            yield line

def parse_logs(lines):
    for line in lines:
        date, time, log = line.split(' ', 2)
        yield date, time, log

filename = "logfile.log"

lines = read_lines(filename)
errors = filter_errors(lines)
logs = parse_logs(errors)

for log in logs:
    print(log)
('2024-06-01', '09:17:42', 'ERROR Unable to connect to database')
('2024-06-02', '09:19:20', 'ERROR Disk space low on drive C:')
('2024-06-04', '09:23:30', 'ERROR Failed to send email notification')
('2024-06-05', '09:27:00', 'ERROR Backup process failed')
```

- Using generators results in improved performance, which is the result of the lazy (on demand) generation of values, which translates to lower memory usage.
- do not need to wait until all the elements have been generated before we start to use them

## recursion

* a function calling itself... 
* where this becomes useful: successive calls are for progressively smaller versions of the problem
* recursive functions have two components: 
  * base case: the simplest chunk of the problem, typically trivial to solve
  * recursive case: the part of the function that calls itself

eg: 5! (5 factorial) is 5 * 4 * 3 * 2 * 1 (120)
- n! = n * (n-1)!
- BASE CASE: 1! = 1 -> can be calculated without performing any more factorial function calls

**note**
The base case acts as the **exit condition** of the recursion

```python

def factorial(n):
    if not isinstance(n, int) or n < 0:
        raise ValueError("Factorial is only defined for non-negative integers.")
    if n in (0, 1):
        return 1
    else:
        return n * factorial(n-1)    


factorial(5)
120
```

```python
def power(x, y):
    if y == 0:
        return 1
    else:
        return x * power(x, y-1)


power(2, 3)
8
```


Recursion can also be indirect.
One function can call a second, which calls the first, which calls the second, and so on.
This can occur with any number of functions

```python
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
### recursion trees & recurrence relations

* recursion trees: a tree that helps visualize the execution foa  recursive function
* recurrence relations: a mathematical representation linking the base and recursive cases

```console
# factorial(5)
# │
# ├─ factorial(4)
# │  │
# │  ├─ factorial(3)
# │  │  │
# │  │  ├─ factorial(2)
# │  │  │  │
# │  │  │  ├─ factorial(1)
# │  │  │  │  │
# │  │  │  │  └─ 1  [Base case]
# │  │  │  │
# │  │  │  └─ 2
# │  │  │
# │  │  └─ 6
# │  │
# │  └─ 24
# │
# └─ 120
```
```python
def recursive_function(n):
    if base_condition_met(n):
        return base_case_value
    else:
        return relation_to_recursive_function(n)
```

### tail recursion & recursion limits

* constant stack space!
  * memory required to store that variables in a function is fixed
* linear stack space
  * linearly proportional to the input size
* tail recursion is a special case of recursion where the recursive call is the last thing executed in the function
* cpython does NOT implement tail call optimization

```python
def factorial(n, accumulator=1):
    if n == 0:
        return accumulator
    else:
        # return n * factorial(n - 1)
        return factorial(n-1, n * accumulator)

factorial(5)
120

import sys

sys.getrecursionlimit()
3000
sys.setrecursionlimit(5000)
```

### mutual recursion

* instead of one function calling itself, two or more functions call each other
* useful when two or more interdependent states alternate with each other
  * e.g. state machines, complex mathemetical sequences

```python
# even -> odd + 1
# odd -> even + 1

def is_even(n):
    if n == 0:
        return True
    else:
        return is_odd(n - 1)
    
def is_odd(n):
    if n == 0:
        return False
    else:
        return is_even(n - 1)

is_odd(7)
True

is_even(10)
True

is_even(1)
False
```
```python
def Q(n):
    if n<=2:
        return 1
    
    return Q(n - Q(n-1)) + Q(n - Q(n-2))

for i in range(1, 10):
    print(Q(i), end=" ")

1 1 2 3 3 4 5 5 6 
```
```console
Q(n) depends Q(n-1) and Q(n-2)
Q(n-1) depends on Q(n-2) and Q(n-3)
Q(n-2) depends on Q(n-3) and Q(n-4)
```
### parsing structured data

```html
<html>
    <body>
        <h1>Heading</h1>
        <p>Paragraph</p>
        <main>
            <p>Another paragraph</p>
        </main>
    </body>
</html>
```

```python
html_string = """<html>
    <body>
        <h1>Heading</h1>
        <p>Paragraph</p>
        <main>
            <p>Another paragraph</p>
        </main>
    </body>
</html>"""

import re

cleaned_html_string = re.sub(r">\s+<", "><", html_string)

# parse_tag -> process the string and identify tags 
# parse_content -> process the content

def parse_tag(s):
    if s.startswith("<"):
        end = s.find(">")
        if end != -1:
            print(f"Found tag: {s[1:end]}")
            return parse_content(s[end+1:])
    return s

def parse_content(s):
    if "<" in s:
        start = s.find("<")
        if start != -1:
            print(f"Content: {s[:start]}")
            return parse_tag(s[start:])
    else:
        print(f"Content: {s}")
    return s

parse_tag(cleaned_html_string)
Found tag: html
Content: 
Found tag: body
Content: 
Found tag: h1
Content: Heading
Found tag: /h1
Content: 
Found tag: p
Content: Paragraph
Found tag: /p
Content: 
Found tag: main
Content: 
Found tag: p
Content: Another paragraph
Found tag: /p
Content: 
Found tag: /main
Content: 
Found tag: /body
Content: 
Found tag: /html
Content: 

''
```

```python
#1 - get rid of empty content output
#2 - get rid of closing tag content

def parse_tag(s):
    if s.startswith("<"):
        end = s.find(">")
        if end != -1:
            tag = s[1:end]
            if not tag.startswith("/"):
                print(f"Found tag: {tag}")
            return parse_content(s[end+1:])
    return s

def parse_content(s):
    if "<" in s:
        start = s.find("<")
        if start != -1:
            content = s[:start].strip() 
            if content:                
                print(f"Content: {content}")
                
            return parse_tag(s[start:])
    else:
        content = s.strip()
        if content:
            print(f"Content: {content}")
    return s

parse_tag(cleaned_html_string)
Found tag: html
Found tag: body
Found tag: h1
Content: Heading
Found tag: p
Content: Paragraph
Found tag: main
Found tag: p
Content: Another paragraph

''
```
### binary search

```console
Implement a recursive function to perform a binary search on a sorted array.

def binary_search(arr, low, high, key):
    # implementation
Parameters:

arr (list): A list of sorted elements (can be numbers or strings).
low (int): The starting index of the array.
high (int): The ending index of the array.
key: The element you are searching for in the array.
Return

The function should return the index of key in arr if key is present. If key is not in the array, return -1.

Constraints:

Assume the array is sorted in ascending order.
The solution must use recursion, not iteration.
Example:

arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

print(binary_search(arr, 0, len(arr) - 1, 5))  # Should return 4
```
```python
def binary_search(arr, low, high, key):
    if high >= low:
        mid = (high + low) // 2 # 5 // 2 -> 2, rounded down
        
        if arr[mid] == key: # mid of arr = key: success!
            return mid
        elif arr[mid] > key: # mid > key -> search the left half of the arr
            return binary_search(arr, low, mid - 1, key)
        else: # mid < key -> search te right half of the arr
            return binary_search(arr, mid + 1, high, key)
    else:
        return -1

arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

print(binary_search(arr, 0, len(arr) - 1, 5))
4

binary_search(arr, 0, len(arr) - 1, 50)
-1

binary_search(arr, 0, len(arr) - 1, 10)
9

binary_search(arr, 0, len(arr) - 1, 1)
0
```
### refactored signature

Refactor the previous binary_search implementation so that it exposes a simpler interface where only arr and key are specified.

The core algorithm should continue to use recursion and does not need much refactoring.

Hint: consider defining a HOF with a simpler signature that wraps around our previous implementation.

```console
# FROM:
def binary_search(arr, low, high, key):
    pass

# TO:
def binary_search(arr, key):
    pass
```
```python
def binary_search(arr, key):
    
    def recursive_search(low, high):
        if high >= low:
            mid = (high + low) // 2 
            
            if arr[mid] == key: 
                return mid
            elif arr[mid] > key: 
                return recursive_search(low, mid - 1)
            else: 
                return recursive_search(mid + 1, high)
        else:
            return -1
    
    return recursive_search(0, len(arr)-1)


arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
binary_search(arr, 0, len(arr) - 1, 5)

binary_search(arr, 5)
4

binary_search(arr, 50)
-1

binary_search(arr, 10)
9

binary_search(arr, 1)
0
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
```python
Accumulate
# reduce() -> takes an iterable and returns a single number, after applying a rolling computation

numbers = [1, 2, 3, 4, 5]

from functools import reduce

sum(numbers)
15

# reduce(
#   func(accumulator, current_item),
#   iterable
# )

reduce(
    lambda acc, x: acc + x,
    numbers
)
15

from functools import reduce

numbers = [1, 2, 3, 4, 5]
sum_of_numbers = reduce(lambda acc, x: acc + x, numbers)
print(sum_of_numbers)  # 15
15

words = ["apple", "banana", "cherry"]

{
    "apple": 5,
    "banana": 6,
    "cherry": 6
}
{'apple': 5, 'banana': 6, 'cherry': 6}

reduce(
    lambda acc, word: acc + word,
    words    
)
'applebananacherry'

reduce(
    lambda acc, word: acc + word,
    words,
    {}
)
TypeError: unsupported operand type(s) for +: 'dict' and 'str'

reduce(
    lambda acc, word: {"word": word},
    words,
    {}
)
{'word': 'cherry'}

reduce(
    lambda acc, word: {word: len(word)},
    words,
    {}
)
{'cherry': 6}

reduce(
    lambda acc, word: {**acc, word: len(word)},
    words,
    {}
)
{'apple': 5, 'banana': 6, 'cherry': 6}
```
```python
dict_list = [{'a': 1}, {'b': 2}, {'c': 3}, {'d': 4}]

from functools import reduce

reduce(
    lambda acc, x: {**acc, **x},
    dict_list
)
{'a': 1, 'b': 2, 'c': 3, 'd': 4}
```
```python
# 5! =
5*4*3*2*1
120

from functools import reduce

def factorial(n):
    return reduce(
        lambda acc, y: acc * y,
        range(1, n+1)
    )

factorial(5)
120
```
use reduce() to calculate the greatest common divisor of a list of integers
```python
numbers = [48, 60, 84]
from math import gcd

numbers = [48, 60, 84]

reduce(gcd, numbers)
12
```
find the average RGB across a list of pixel tuples using reduce() and map()
```python
pixels = [(120, 200, 255), (110, 190, 220), (100, 180, 210), (130, 210, 230)]

tuple(map(
    lambda x: x // len(pixels),
    reduce(
        lambda acc, y: tuple(xi + yi for xi, yi in zip(acc, y)),
        pixels
    )
))
(115, 195, 228)
```
run-length encoding (RLE) is a basic form of lossless data compression where sequences of identical data values (runs) are stored as a single data value and count

```python
# assert run_length_encode("AABCCDEEEE") == "A2B1C2D1E4"
# AABCCDEEEE -> ("A", 2), ("B", 1), ("C", 2)... -> A2B1C2...

from functools import reduce

def run_length_encode(s):
    def reducer(acc, char):
        if acc and char == acc[-1][0]:
            acc[-1] = (char, acc[-1][1] + 1)
        else:
            acc.append((char, 1))
        
        return acc
    
    # encoded string
    encoded_pairs = reduce(reducer, s, []) # -> [("A", 2), ("B", 1), ("C", 2)...]
    
    encoded_str = "".join(f"{char}{count}" for char, count in encoded_pairs)
    
    return encoded_str

run_length_encode("AABCCDEEEEEPOOPPPPDODOOPPP")
'A2B1C2D1E5P1O2P4D1O1D1O2P3'
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

## decorators

add logic to function without changing it

```python
def fry():
    return "Frying the food"

def grill():
    return "Grilling the food"

def boil():
    return "Boiling the food"

def seasoning(chef):
    def wrapper():
        print("Adding some salt and pepper")
        return chef()
    return wrapper

@seasoning
def fry():
    return "Frying the food"

@seasoning
def grill():
    return "Grilling the food"

@seasoning
def boil():
    return "Boiling the food"

fry()
Adding some salt and pepper

'Frying the food'

def fry():
    return "Frying the food"

seasoned_fry = seasoning(fry)

seasoned_fry()
Adding some salt and pepper

'Frying the food'
```

### timer

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

using perf counter

```python
from time import perf_counter

def timed(func):
    def wrapper(*args, **kwargs):
        start_time = perf_counter()        
        result = func(*args, **kwargs)
        end_time = perf_counter()
        
        print(f"Function {func.__name__} took {round(end_time - start_time, 4)} seconds to execute.")
        return result
        
    return wrapper

@timed
def loop_this_many_times(n):
    for i in range(n):
        pass
    return "testing"

loop_this_many_times(10**5)
Function loop_this_many_times took 0.0032 seconds to execute.

'testing'

loop_this_many_times(10**6)
Function loop_this_many_times took 0.0426 seconds to execute.


loop_this_many_times(10**7)
Function loop_this_many_times took 0.3375 seconds to execute.


loop_this_many_times(10**8)
Function loop_this_many_times took 4.5683 seconds to execute.
```

### backoff

```python
import backoff
import requests
# https://requests.readthedocs.io/en/latest/user/quickstart/
# https://pypi.org/project/backoff/


def header() -> dict[str, str]:
    hdr = {
        'Accept': 'application/json',
        'Accept-Encoding': 'deflate',
        'User-Agent': 'Mozilla/5.0'
    }
    return hdr


@backoff.on_exception(backoff.expo,
                      (requests.exceptions.Timeout,
                       requests.exceptions.ConnectionError),
                      max_time=30)
def request_json(url: str, payload: dict, custom_header: dict):
    try:
        response = requests.get(url, params=payload, headers=custom_header)
        result = response.json()
    except requests.HTTPError as http_err:
        print(f"request failed with HTTPError: {http_err}")
    except requests.JSONDecodeError as json_err:
        print(f"couldn't decode request's response into json: {json_err}")
    except Exception as err:
        print(f"Request failed with exception: {err}")
    else:
        # code here is executed when there is no error
        return result
```

### advanced decorators

```python
def calories_burned(duration_in_minutes, calories_burned_per_minute):
    return duration_in_minutes * calories_burned_per_minute

calories_burned(30, 10)
300

def ensure_healthy_workout(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        if result < 500:
            print("This workout is not intense enough.")
        else:
            print(f"Well done. Target exceeded by {result-500} calories.")
            
    return wrapper

@ensure_healthy_workout
def calories_burned(duration_in_minutes, calories_burned_per_minute):
    return duration_in_minutes * calories_burned_per_minute

calories_burned(30, 10)
This workout is not intense enough.


calories_burned(60, 10)
Well done. Target exceeded by 100 calories.


def ensure_healthy_workout(calorie_target):    
    def actual_decorator(func):
        def wrapper(*args, **kwargs):
            result = func(*args, **kwargs)
            if result < calorie_target:
                print("This workout is not intense enough.")
            else:
                print(f"Well done. Target exceeded by {result-calorie_target} calories.")
                
        return wrapper
    return actual_decorator

@ensure_healthy_workout(calorie_target=300)
def calories_burned(duration_in_minutes, calories_burned_per_minute):
    return duration_in_minutes * calories_burned_per_minute

calories_burned(40, 10)
Well done. Target exceeded by 100 calories.
```

```python
def add(x, y):
    return x+y

add.__name__
'add'

def trace(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with arguments {args} and keyword arguments {kwargs}")
        original_result = func(*args, **kwargs)
        print(f"Function {func.__name__} returned {original_result}")
        return original_result
    return wrapper

add(3,4)
7

trace_add = trace(add)

trace_add(3,4)
Calling add with arguments (3, 4) and keyword arguments {}
Function add returned 7
7
```

### chaining decorators

```python
def passphrase():
    return "Horizontal Omit Station Reflection"

# splits
# upperscase

def uppercase(func):
    def wrapper():
        result = func()
        return result.upper()
    return wrapper

def split(func):
    def wrapper():
        result = func()
        return result.split()
    return wrapper

@split
@uppercase
def passphrase():
    return "Horizontal Omit Station Reflection"

passphrase()
['HORIZONTAL', 'OMIT', 'STATION', 'REFLECTION']

# @uppercase
# "Horizontal Omit Station Reflection" -> 'HORIZONTAL OMIT STATION REFLECTION'
# @split
# 'HORIZONTAL OMIT STATION REFLECTION' -> ['HORIZONTAL', 'OMIT', 'STATION', 'REFLECTION']

@uppercase
@split
def passphrase():
    return "Horizontal Omit Station Reflection"

passphrase()
AttributeError: 'list' object has no attribute 'upper'

"Horizontal Omit Station Reflection".split()
['Horizontal', 'Omit', 'Station', 'Reflection']

# @split
# "Horizontal Omit Station Reflection" -> ['Horizontal', 'Omit', 'Station', 'Reflection']

# @uppercase
# ['Horizontal', 'Omit', 'Station', 'Reflection'].upper() -> TypeError

['Horizontal', 'Omit', 'Station', 'Reflection'].upper()
AttributeError: 'list' object has no attribute 'upper'
```

### wraps, preserve id

```python
def passphrase():
    """Returns a string."""
    return "Horizontal Omit Station Reflection"

passphrase
<function __main__.passphrase()>

passphrase.__name__
'passphrase'

passphrase.__doc__
'Returns a string.'

def split(func):
    def wrapper():
        result = func()
        return result.split()
    return wrapper

@split
def passphrase():
    """Returns a string."""
    return "Horizontal Omit Station Reflection"

passphrase()
['Horizontal', 'Omit', 'Station', 'Reflection']

passphrase.__name__
'wrapper'

passphrase.__doc__

from functools import wraps

def split(func):
    @wraps(func)
    def wrapper():
        result = func()
        return result.split()
    return wrapper

@split
def passphrase():
    """Returns a string."""
    return "Horizontal Omit Station Reflection"

passphrase()
['Horizontal', 'Omit', 'Station', 'Reflection']

passphrase.__name__
'passphrase'

passphrase.__doc__
'Returns a string.'

passphrase.__wrapped__.__name__
'passphrase'

passphrase.__name__
'passphrase'

passphrase.__wrapped__.__name__ is passphrase.__name__
True
# copy operation!
```

### practice

- define a decorator called delay_decorator
- the decorator should be applicable to a function that takes user_id and resource as parameters.
- keep track of the delays for each user. Initially, a user may have no delay, but the delay should increase with each function call by that user
- on each call, the delay for that user should double, starting from zero (0, 1, 2, 4, 8 seconds, etc.)
- before the function executes, print a message indicating the delay: "Your download will start in [delay]s"
- after the delay, the decorated function should execute normally. Ensure that the original function’s return value is not affected by the decorator
- to test, create a function download(user_id, resource) that simulates a resource download and returns a placeholder download link. Apply your delay_decorator to the download function

sample output
```
print(download("user123", "resource1"))
# Your resource is ready at: domain.com/c26689f7-1f59-4708-9626-dd40a02b0fc8

print(download("user123", "resource2"))
# Your download will start in 1s...
# Your resource is ready at: domain.com/d2caa6fa-55c3-492a-9c7e-fd451778c2df

print(download("user123", "resource2"))
# Your download will start in 2s...
# Your resource is ready at: domain.com/ec9b2847-e5f0-4b9f-92e6-d8b0f0100962
```

```python
from uuid import uuid4

def download(user_id, resource):
    download_uuid = uuid4()
    
    download_url = f"domain.com/{download_uuid}"
    return f"Your resource is ready at: {download_url}"

print(download("user123", "resource1"))

print(download("user123", "resource2"))

print(download("user123", "resource2"))
Your resource is ready at: domain.com/7258252a-e4c4-46f5-8a21-12e34041fac0
Your resource is ready at: domain.com/ad75acfc-862d-465a-9a6f-f2b7731d6d4d
Your resource is ready at: domain.com/88d22db7-57da-4fb1-8def-421f6e8f61e1


import time
from functools import wraps

def delay_decorator(func):
    user_delay = {}
    
    @wraps(func)
    def wrapper(user_id, resource):
        # what is the applicable delay for a given user_id?
        delay = user_delay.get(user_id, 0)
        user_delay[user_id] = max(1, delay * 2)
        
        if delay > 0:
            print(f"Your download will start in {delay} seconds")
            
        time.sleep(delay)
        return func(user_id, resource)
    
    return wrapper

from uuid import uuid4

@delay_decorator
def download(user_id, resource):
    download_uuid = uuid4()
    
    download_url = f"domain.com/{download_uuid}"
    return f"Your resource is ready at: {download_url}"

print(download("user123", "resource1"))

print(download("user123", "resource2"))

print(download("user123", "resource2"))
Your resource is ready at: domain.com/40ca041b-26ef-4a2d-9929-9f6340cae831
Your download will start in 1 seconds
Your resource is ready at: domain.com/40e4af11-13b1-4ad9-ae90-0b21f2602cbb
Your download will start in 2 seconds
Your resource is ready at: domain.com/00b9942d-e06e-4e33-9f6f-c4301c4f74e3


print(download("user12asdfasdf3", "resource2"))
Your resource is ready at: domain.com/cb62d4c2-1d3f-4d06-aa1e-3fb62967a212


print(download("user12=asdasdfasdf3", "resource2"))
Your resource is ready at: domain.com/ac2fffcb-e00e-4b94-b8e5-8bc6b3c48e18


print(download("user12=asdasdfasdf3", "resource2"))
Your download will start in 1 seconds
Your resource is ready at: domain.com/05c7e397-5716-49a1-906e-1f3f5c578012


print(download("user12=asdasdfasdf3", "resource2"))
print(download("user12=asdasdfasdf3", "resource2"))
print(download("user12=asdasdfasdf3", "resource2"))
print(download("user12=asdasdfasdf3", "resource2"))
print(download("user12=asdasdfasdf3", "resource2"))
Your download will start in 2 seconds
Your resource is ready at: domain.com/d28c5d4d-39a5-4710-91b6-7a997b446a8c
Your download will start in 4 seconds
Your resource is ready at: domain.com/ecf5ed1f-9f53-46ad-8aca-fb64fa3faedb
Your download will start in 8 seconds
Your resource is ready at: domain.com/e45fad9d-a81e-4821-91dd-b7e0e4799989
Your download will start in 16 seconds
Your resource is ready at: domain.com/fe7f4096-01cc-47bd-8547-6f6ce609460b
Your download will start in 32 seconds
Your resource is ready at: domain.com/8db56f70-5059-42ba-81de-a9f516d17962
```
