# core

## time & dates

### timezones

example of UTC & Chicago time (CST) usage

```python
from datetime import datetime, timezone
import pytz

naive_dt = datetime.now()
2024-01-12 13:41:11.271064
    
naive_utc_dt = datetime.utcnow()
2024-01-12 12:41:11.271073
    
utc_dt = datetime.now(timezone.utc)
2024-01-12 12:41:11.271077+00:00
    
dt = utc_dt.astimezone()  # localtime
2024-01-12 13:41:11.271077+01:00

tz = pytz.timezone('America/Chicago')  # CST
America/Chicago
    
fed_now = datetime.now(tz)
2024-01-12 06:41:11.301733-06:00
```

## itertools

list of [iteration tools](https://docs.python.org/3/library/itertools.html) in python

### accumulate

```python
from itertools import accumulate, takewhile

nums = list(accumulate(range(8)))
"""[0,  1,   3,   6,  10,   15,   21, 28]"""
```

### count

```python
from itertools import count

for i in count(3): # counts up starting from 3
	print(i)
	if i>= 11:
		break

""" 3  4  5  6  7  8  9  10  11 """
```

### islice

```python
from itertools import islice, count
from math import sqrt


def is_prime(x):
    if x < 2:
        return False
    for i in range(2, int(sqrt(x)) + 1):
        if x % i == 0:
            return False
    return True

# do this thousand_primes = islice(all_primes, 1000) but how to generate all primes
# ranges must always be finite, we need an open ended version of range and that is what count() does
# thousand_primes = islice((x for x in count() if is_prime(x)), 1000) # with islice() like with lists

sum(islice((x for x in count() if is_prime(x)), 1000))
3682913
```

### chain

syncronize iterations over 2 iterable series
eg two  series of temperature data

```python
sunday = [12, 14, 15, 15, 17, 21, 22, 22, 23, 22, 20, 18]
monday = [13, 14, 14, 14, 16, 20, 21, 22, 22, 21, 19, 17]
# bind them in pairs of corresponding readings
for item in zip(sunday, monday):
    print(item)

(12, 13)
(14, 14)
(15, 14)
(15, 14)
(17, 16)
(21, 20)
(22, 21)
(22, 22)
(23, 22)
(22, 21)
(20, 19)
(18, 17)

# zip yields tuples when iterated
# we can take advantage of this with tuple unpacking in the for loop
for sun, mon in zip(sunday, monday):
    print("average =", (sun + mon) / 2)

average = 12.5
average = 14.0
average = 14.5
average = 14.5
average = 16.5
average = 20.5
average = 21.5
average = 22.0
average = 22.5
average = 21.5
average = 19.5
average = 17.5

tuesday = [2, 2, 3, 7, 9, 10, 9, 8, 8]

for temps in zip(sunday, monday, tuesday):
    print("min={:4.1f}, max={:4.1f}, average={:4.1f}".format(min(temps), max(temps), sum(temps) / len(temps)))
""" 
min= 2.0, max=13.0, average= 9.0
min= 2.0, max=14.0, average=10.0
min= 3.0, max=15.0, average=10.7
min= 7.0, max=15.0, average=12.0
min= 9.0, max=17.0, average=14.0
min=10.0, max=21.0, average=17.0
min= 9.0, max=22.0, average=17.3
min= 8.0, max=22.0, average=17.3
min= 8.0, max=23.0, average=17.7
"""
# now we want one long temperature series for sunday monday and thuesday 

# we can then lazily concatenate iterables using itertools chain

# this is different from simply concatenating 3 lists into a new list

# we have no memory impact of data duplication
from itertools import chain
temperatures = chain(sunday, monday, tuesday)

all(t > 0 for t in temperatures)
temperatures = chain(sunday, monday, tuesday)
True

# following shows generator functions, generator expressions, predicate functions and for loops
def lucas():
    yield 2
    a = 2
    b = 1
    while True: # infinite while loop
        yield b
        a, b = b, a + b

for x in (p for p in lucas() if is_prime(p)):
    print(x)

2
3
7
11
29
47
199
521
2207
3571
9349
3010349
54018521
370248451
6643838879
119218851371
5600748293801
688846502588399
32361122672259149
```


```python
itertools.chain(*iterables)
Make an iterator that returns elements from the first iterable until it is exhausted, then proceeds to the next iterable, until all of the iterables are exhausted. Used for treating consecutive sequences as a single sequence
```

https://docs.python.org/3/library/itertools.html#itertools.chain

### take

take generator

```python
def take(count, iterable):
    """Take items from the front of an iterable.

    Args:
        count: maximum number of items to retrieve
        iterable: the source series

    Yields:
         at most 'count' items from 'iterable'
    """

    counter = 0
    for item in iterable:
        if counter == count:
            return # end sequence when we reach specified count
            # return raises StopIteration which is caught internally by the for loop in run_take()
        counter += 1 # how many items have been yielded so far
        yield item # contains a generator bc it has at least one yield statement


def run_take(): # generators are lazy and only generate values on request
    items = [2, 4, 6, 8, 10]
    for item in take(3, items):  # take(count, iterable) # return raises StopIteration which is caught by
        print(item)


if __name__ == "__main__":
        run_take()
```

### takewhile/dropwhile

```python
print(list(takewhile(lambda x: x<=6, nums)))  # [0, 1, 3, 6]

# takewhile stops as soon as predicate == FALSE!
nums = [2, 4, 6, 7, 9, 8]  # will stop returning at hitting value 7

print(list(takewhile(lambda x: x%2==0, nums))) 
# [2, 4, 6]
```

```python
import itertools

def testFunction(x):
    return x < 40
    
# dropwhile and takewhile will return values until
# a certain condition is met that stops them
vals = [10,20,30,40,50,40,30]
print(list(itertools.dropwhile(testFunction, vals)))
""" [40, 50, 40, 30] """
print(list(itertools.takewhile(testFunction, vals)))
""" [10, 20, 30] """
```

### product & permutations

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

### combinations

```python

import itertools

words = sorted({line.strip().lower() for line in open('/usr/share/dict/words', 'r')})


def signature(word):
    return ''.join(sorted(word))


words_by_sig = collections.defaultdict(set)


for word in words:
    words_by_sig[signature(word)].add(word)

    
anagrams_by_sig = {sig: wordset for sig, wordset in words_by_sig.items() if len(wordset) > 1}


# list all combinations of two different elements from the set {1,2,3} 
list(itertools.combinations({1,2,3}, 2))
[(1, 2), (1, 3), (2, 3)]

# same as above, using itertools.combinations to generate unique pairs

pairs = []

for wordset in anagrams_by_sig.values():
    for word1, word2 in itertools.combinations(wordset, 2):
        if word1[::-1] == word2:
            pairs.append((word1, word2))

pairs
[('ba', 'ab'), ('able', 'elba'), ('mba', 'abm'), ('abut', 'tuba'), ('ac', 'ca'), ('ad', 'da'),
('fa', 'af'), ('ga', 'ag'), ('agar', 'raga'), ('sega', 'ages'), ('ha', 'ah'), ('ia', 'ai'),
('ila', 'ali'), ('delia', 'ailed'), ('mia', 'aim'), ('iras', 'sari'), ('ajar', 'raja'), ('la', 'al')
..
('zeus', 'suez'), ('sway', 'yaws'), ('tort', 'trot'), ('wot', 'tow'), ('vt', 'tv'), ('way', 'yaw')]
```

### batched

```python
flattened_data = ['roses', 'red', 'violets', 'blue', 'sugar', 'sweet']
unflattened = list(batched(flattened_data, 2))
unflattened
[('roses', 'red'), ('violets', 'blue'), ('sugar', 'sweet')]

for batch in batched('ABCDEFG', 3):
    print(batch)

('A', 'B', 'C')
('D', 'E', 'F')
('G',)
```

Roughly equivalent to

```python
def batched(iterable, n):
    # batched('ABCDEFG', 3) --> ABC DEF G
    if n < 1:
        raise ValueError('n must be at least one')
    it = iter(iterable)
    while batch := tuple(islice(it, n)):
        yield batch
```

## collections

This module implements specialized container datatypes providing alternatives to Python's general purpose built-in containers, dict, list, set, and tuple.

https://docs.python.org/3/library/collections.html

|collection|description|
|--|--|
|namedtuple|	factory function for creating tuple subclasses with named fields|
|deque|	list-like container with fast appends and pops on either end|
|ChainMap|	dict-like class for creating a single view of multiple mappings|
|Counter|	dict subclass for counting hashable objects|
|OrderedDict|	dict subclass that remembers the order entries were added|
|defaultdict|	dict subclass that calls a factory function to supply missing values|
|UserDict|	wrapper around dictionary objects for easier dict subclassing|
|UserList|	wrapper around list objects for easier list subclassing|
|UserString|	wrapper around string objects for easier string subclassing|

### namedtuple

```python
import pprint
from collections import namedtuple
import statistics as st


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

summary(mean=6.5, median=7.0, mode=7)
```

```python
people = [("Michele", "Vallisneri", "July 15"),
          ("Albert", "Einstein", "March 14"),
          ("John", "Lennon", "October 9"),
          ("Jocelyn", "Bell Burnell", "July 15")]

people[0][0]
'Michele'
people[0][1]
'Vallisneri'

[person for person in people if person[2] == "July 15"]
[('Michele', 'Vallisneri', 'July 15'), ('Jocelyn', 'Bell Burnell', 'July 15')]

# defining the namedtuple "person"
persontype = collections.namedtuple('person', ['firstname', 'lastname', "birthday"])
michele = persontype("Michele", "Vallisneri", "July 15")

michele = persontype(lastname="Vallisneri", firstname="Michele", birthday="July 15")
michele
person(firstname='Michele', lastname='Vallisneri', birthday='July 15')

michele[0], michele[1], michele[2]
('Michele', 'Vallisneri', 'July 15')

michele.firstname, michele.lastname, michele.birthday
('Michele', 'Vallisneri', 'July 15')

# use tuple unpacking on people[0] to build a namedtuple
persontype(*people[0])
person(firstname='Michele', lastname='Vallisneri', birthday='July 15')

namedpeople = [persontype(*person) for person in people]
namedpeople
[person(firstname='Michele', lastname='Vallisneri', birthday='July 15'), person(firstname='Albert', lastname='Einstein', birthday='March 14'), person(firstname='John', lastname='Lennon', birthday='October 9'), person(firstname='Jocelyn', lastname='Bell Burnell', birthday='July 15')]

[person for person in namedpeople if person.birthday == "July 15"]
[person(firstname='Michele', lastname='Vallisneri', birthday='July 15'), person(firstname='Jocelyn', lastname='Bell Burnell', birthday='July 15')]
```

```python
Point = collections.namedtuple("Point", "x y")

p1 = Point(10, 20)
p2 = Point(30, 40)

print(p1, p2)
print(p1.x, p1.y)

# _replace to create a NEW instance
p1 = p1._replace(x=100)
print(p1)
```

```python
"""
print(open("goldmedals.txt", "r").readlines()[:10])
'1896\tThomas Burke\tUSA\t100m men\n',
'1896\tThomas Curtis\tUSA\t110m hurdles men\n',
"""
# tuple named Medal
Medal = collections.namedtuple('Medal', ['year', 'athlete', 'team', 'event'])
# medals = [line.split('\t') for line in open("goldmedals.txt", "r")]
# fill list "medals" with list of Medal namedtuples
medals = [Medal(*line.strip().split('\t')) for line in open("files/goldmedals.txt", "r")]
"""
Medal(year='1896', athlete='Thomas Burke', team='USA', event='100m men'),
Medal(year='1896', athlete='Thomas Curtis', team='USA', event='110m hurdles men'),
"""
teams = collections.Counter(medal.team for medal in medals)
Counter({'USA': 521,
         'AUS': 23,
         'GRE': 7,
         'GBR': 71,
         'CAN': 21,
         'ZZX': 5,
			..
         'SWE': 21,
         'SUI': 1,
         'COL': 1,
         'BRN': 1,
         'TJK': 1,
         'SVK': 1})

best = collections.Counter(medal.team for medal in medals if medal.year == str(2016)).most_common(5)
best
[('USA', 30), ('JAM', 11), ('KEN', 6), ('GBR', 3), ('GER', 2)]

def best_by_year(year):
    counts = collections.Counter(medal.team for medal in medals if medal.year == str(year))
    best = counts.most_common(5)
    # [('USA', 30), ('JAM', 11), ('KEN', 6), ('GBR', 3), ('GER', 2)]
    return [x[0] for x in best], [y[1] for y in best]

[52]

best_by_year(2016)
(['USA', 'JAM', 'KEN', 'GBR', 'GER'], [30, 11, 6, 3, 2])
```

### defaultdict

if key doesn't exist yet create one instead of throwing an error
takes factory function as argument so can be a custom lambda too
like `defaultdict(lambda: 33)`

don't use if a missing key in a dict is really important

```python
>>> test = defaultdict(lambda: 100/2)
>>> test
defaultdict(<function <lambda> at 0x7f36b62f2320>, {})
>>> test["nonexistent"] += 1
>>> test
defaultdict(<function <lambda> at 0x7f36b62f2320>, {'nonexistent': 51.0})
>>> 
```

```python
from collections import defaultdict

fruits = ['apple', 'pear', 'orange', 'banana',
          'apple', 'grape', 'banana', 'banana']

# use a dictionary to count each element
fruitCounter = defaultdict(int)


# count the elements in the list
for fruit in fruits:
    fruitCounter[fruit] += 1

# print the result
for (k, v) in fruitCounter.items():
    print(k + ": " + str(v))
```

### ordereddict

```python
from collections import OrderedDict

# list of sport teams with wins and losses
sportTeams = [("FlyingPenguins", (18, 12)), ("Rabbits", (24, 6)), 
            ("DarkKnights", (20, 10)), ("Zombies", (22, 8)),
            ("MightyMisfits", (15, 15)), ("Ducks", (20, 10)), 
            ("TeamTango", (16, 14)), ("Warriors", (25, 5))]

# sort the teams by number of wins
sortedTeams = sorted(sportTeams, key=lambda t: t[1][0], reverse=True)

# create an ordered dictionary of the teams
teams = OrderedDict(sortedTeams)
print(teams)

# use popitem to remove the top item
tm, wl = teams.popitem(False)
print("Top team: ", tm, wl)

# what are the next top 4 teams?
for i, team in enumerate(teams, start=1):
    print(i, team)
    if i == 4:
        break

# test for equality
a = OrderedDict({"a": 1, "b": 2, "c": 3})
b = OrderedDict({"a": 1, "c": 3, "b": 2})
print("Equality test: ", a == b)
```

### chainmap

```python
>>> baseline = {'music': 'bach', 'art': 'rembrandt'}
>>> adjustments = {'art': 'van gogh', 'opera': 'carmen'}
>>> list(ChainMap(adjustments, baseline))
['music', 'art', 'opera']
```

### counter

```python
# list of students in class 1
class1 = ["Rob", "Robert", "Chad", "Darcy", "Penny", "Henry"
          "Kevin", "Robert", "Melissa", "Becky", "Steve", "Flynn"]

# list of students in class 2
class2 = ["Bill", "Barry", "Cindy", "Debbie", "Flynn",
          "Gabby", "Kelly", "Robert", "Joe", "Sam", "Tara", "Ziggy"]

# Create a Counter for class1 and class2
c1 = Counter(class1)
c2 = Counter(class2)

# How many students in class 1 named Robert?
print(c1["Robert"])

# How many students are in class 1?
print(sum(c1.values()), "students in class 1")

# Combine the two classes
c1.update(class2)
print(sum(c1.values()), "students in class 1 and 2")

# What's the most common name in the two classes?
print(c1.most_common(3))

# Separate the classes again
c1.subtract(class2)
print(c1.most_common(1))

# What's common between the two classes?
print(c1 & c2)
```

### deque

double ended queue

```python
import collections
import string

# initialize a deque with lowercase letters
d = collections.deque(string.ascii_lowercase)

# deques support the len() function
print("Item count: " + str(len(d)))

# deques can be iterated over
for elem in d:
    print(elem.upper(), end=",")

# manipulate items from either end
d.pop()
d.popleft()
d.append(2)
d.appendleft(1)
print(d)

# rotate the deque
print(d)
d.rotate(1)
print(d)
```

## typing

```python
from typing import Any, Dict, Literal, Union

import pandas as pd

from pyfredapi.utils.enums import ReturnFormat

ApiKeyType = Union[str, None]
JsonType = Dict[str, Any]
JsonOrPdType = Union[Dict, pd.DataFrame]
ReturnFmtType = Union[Literal["json", "pandas"], ReturnFormat]
KwargsType = Dict[str, Union[int, str, None]]

```
ReturnFormat class

```python
from enum import Enum


class ReturnFormat(str, Enum):
    """Defines how to format the data returned from the FRED endpoint."""

    pandas = "pandas"
    json = "json"
```

### optional/literal

```python
class SeriesApiParameters(BaseModel):

    series_id: Optional[str] = None
    realtime_start: Optional[str] = None
    realtime_end: Optional[str] = None
    limit: Optional[int] = None
    offset: Optional[PositiveInt] = None
    sort_order: Optional[Literal["acs", "desc"]] = None
    observation_start: Optional[str] = None
    observation_end: Optional[str] = None
    units: Optional[
        Literal[
            "lin",
            "chg",
            "ch1",
            "pch",
            "pc1",
            "pca",
            "cch",
            "cca",
            "log",
        ]
    ] = None
    frequency: Optional[
        Literal[
            "d",
            "w",
            "bw",
            "m",
            "q",
            "sa",
            "a",
            "wef",
            "weth",
            "wew",
            "wetu",
            "wem",
            "wesu",
            "wesa",
            "bwew",
            "bwem",
        ]
    ] = None
    aggregation_method: Optional[Literal["avg", "sum", "eop"]] = None
    output_type: Optional[Literal["1", "2", "3", "4", 1, 2, 3, 4]] = None
    vintage_dates: Optional[str] = None
```
