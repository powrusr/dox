# performance

## timeit

```python
import sys
import timeit
from functools import partial

NUM_REPEATS = 100


def convert_unit(value):
    for unit in ["s", "ms", "μs", "ns"]:
        if value > 1:
            return f"{value:8.2f}{unit}"
        value *= 1000


def timer(func):
    def _timer(*args, **kwargs):
        tictoc = timeit.timeit(
            partial(func, *args, **kwargs), number=NUM_REPEATS
        )
        print(f"{func.__name__:<20} {convert_unit((tictoc) / NUM_REPEATS)}")

    return _timer


@timer
def plain(numbers):
    return [number for number in numbers]


@timer
def square(numbers):
    return [number**2 for number in numbers]


@timer
def filtered(numbers):
    return [number for number in numbers if number % 2 == 0]


n, N = 100, 100_000
small_numbers = range(n)
big_numbers = range(N)
repeated_numbers = [1 for _ in range(N)]

print(sys.version)

print("\nOne item iterable:")
plain([0])
square([0])
filtered([0])

print(f"\nSmall iterable ({n = :,}):")
plain(small_numbers)
square(small_numbers)
filtered(small_numbers)

print(f"\nBig iterable ({N = :,}):")
plain(big_numbers)
square(big_numbers)
filtered(big_numbers)

print(f"\nRepeated iterable ({N = :,}):")
plain(repeated_numbers)
square(repeated_numbers)
filtered(repeated_numbers)
```

## Least Recently Used

```python
from functools import lru_cache

@lru_cache
def fib(n):
    if n <= 1:
        return 1
    return fib(n-1) + fib(n-2)

for i in range(25, 40):
    elapsed = timeit(f"fib({i})", globals=globals(), number=1)
    print(f"{i} - {elapsed} s")
```
```
25 - 1.2166041415184736e-05 s
26 - 4.169996827840805e-07 s
27 - 2.92027834802866e-07 s
28 - 2.919696271419525e-07 s
29 - 2.9103830456733704e-07 s
30 - 2.500019036233425e-07 s
31 - 2.500019036233425e-07 s
32 - 2.0797597244381905e-07 s
33 - 2.500019036233425e-07 s
34 - 2.500019036233425e-07 s
35 - 2.9098009690642357e-07 s
36 - 2.0803418010473251e-07 s
37 - 2.0902371034026146e-07 s
38 - 2.500019036233425e-07 s
39 - 2.0797597244381905e-07 s
```

unlimited
```python
@lru_cache(maxsize=None)
def fib(n):
    if n <= 1:
        return 1
    return fib(n-1) + fib(n-2)
```

limited

if you look at the code closely, and the trace outputs we had earlier, you'll realize that in fact we only need to ever cache the last 2 calls - so we could limit our LRU cache to just two elements, without losing the speedup benefits:

```
@lru_cache(maxsize=2)
def fib(n):
    if n <= 1:
        return 1
    return fib(n-1) + fib(n-2)  
for i in range(25, 40):
    elapsed = timeit(f"fib({i})", globals=globals(), number=1)
    print(f"{i} - {elapsed} s")
```
```
25 - 0.000801375019364059 s
26 - 0.00030200002947822213 s
27 - 0.0004160410026088357 s
28 - 0.0005925829755142331 s
29 - 0.00079158297739923 s
30 - 0.0010942919761873782 s
31 - 0.0016196249634958804 s
32 - 0.002049707982223481 s
33 - 0.0027969160000793636 s
34 - 0.00412495800992474 s
35 - 0.005773082957603037 s
36 - 0.007761000015307218 s
37 - 0.01030524994712323 s
38 - 0.013624083949252963 s
39 - 0.018959375040140003 s
```

### unbounded cache
instead of using `@lru_cache(maxsize=None)` you can use `@cache`

```python 
from functools import cache

@cache
def fib(n):
    if n <= 1:
        return 1
    return fib(n-1) + fib(n-2)
```

### cached property

```python
from functools import cached_property

class Circle():
    def __init__(self, radius):
        self._radius = radius
        self._area = None

    @cached_property
    def area(self):
        return pi * (self._radius ** 2)


c = Circle(3)

timeit("c.area", globals=globals(), number=1_000_000)
0.024079292023088783
```

Caveats of Cached Properties and Mutability
> Now, this approach here works because our Circle class is immutable (by convention). We assume that since we have set our radius to be private (by convention, using that leading underscore), that the radius will never change over the lifetime of our Circle(3) instance

> If the radius does change for some reason, we'll run into issues (both with our own approach, and with the cached_property approach.

#### make property mutable

This was our original implementation:

```python
class Circle():
    def __init__(self, radius):
        self._radius = radius

    @cached_property
    def area(self):
        return pi * (self._radius ** 2)
```

Let's add the radius property and invalidate the cache as needed in the radius setter, just like we did in our first principle example:
```python
class Circle():
    def __init__(self, radius):
        self._radius = radius
        self._area = None

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if self._radius != value:
            del self.area
        self._radius = value
        
    @cached_property
    def area(self):
        return pi * (self._radius ** 2)


c = Circle(1)
c.area
3.141592653589793

c.radius = 2
c.area
12.566370614359172
```
As you can see, we get the same effect now

### caching class methods

cached_property establishes a cache at the instance level, caching a method using cache or lru_cache establishes a cache at the class level

Make sure cache picks up on objects no longer being equal with `__eq__` and `__hash__`

```python
class Circle:
    def __init__(self, r):
        self.r = r

    @cache
    def area(self):
        print(f"calculating area for {self.r=}")
        return pi * (self.r ** 2)

    def __eq__(self, other):
        return self.r == other.r

    def __hash__(self):
        return hash(self.r)

c1 = Circle(1)
c2 = Circle(1)

c1 == c2
True
```

## cachetools

- [cache tools github](https://github.com/tkem/cachetools/)
- [cache tools docs](https://cachetools.readthedocs.io/en/latest/)

```python
from cachetools import cached, LRUCache, TTLCache

# speed up calculating Fibonacci numbers with dynamic programming
@cached(cache={})
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)

# cache least recently used Python Enhancement Proposals
@cached(cache=LRUCache(maxsize=32))
def get_pep(num):
    url = 'http://www.python.org/dev/peps/pep-%04d/' % num
    with urllib.request.urlopen(url) as s:
        return s.read()

# cache weather data for no longer than ten minutes
@cached(cache=TTLCache(maxsize=1024, ttl=600))
def get_weather(place):
    return owm.weather_at_place(place).get_weather()
```

