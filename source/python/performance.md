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
## caching

### manual cache

```python
import icecream
import json
import requests


def fetch_data(*, update: bool = False, json_cache: str, url: str):  # * = named arguments only
    if update:
        json_data = None
    else:
        try:
            with open(json_cache, 'r') as f:
                json_data = json.load(f)
                print("data loaded from cache")
        except (FileNotFoundError, json.JSONDecodeError) as e:
            print(f"No local cache found: {e}")
            json_data = None

    if not json_data:
        print("Fetching new json data.. (creating local cache)")
        json_data = requests.get(url).json()
        with open(json_cache, "w") as f:
            json.dump(json_data, f)

    return json_data


if __name__ == "__main__":
    url_comments = "http://dummyjson.com/comments"
    cache_file = "comments.json"
    # fetch_data(update=True)
    data: dict = fetch_data(update=False, json_cache=cache_file, url=url_comments)
    icecream.ic(data)
"""
No local cache found: [Errno 2] No such file or directory: 'comments.json'
Fetching new json data.. (creating local cache)
ic| data: {'comments': [{'body': 'This is some awesome thinking!',
                         'id': 1,
                         'postId': 100,
                         'user': {'id': 63, 'username': 'eburras1q'}},
                        {'body': 'What terrific math skills you’re showing!',
                         'id': 2,
                         'postId': 27,
                         'user': {'id': 71, 'username': 'omarsland1y'}},
                        {'body': 'You are an amazing writer!',
                         'id': 3,
                         'postId': 61,
                         'user': {'id': 29, 'username': 'jissetts'}},
                         ...
                        {'body': 'This is very perceptive!',
                         'id': 29,
                         'postId': 13,
                         'user': {'id': 30, 'username': 'kdulyt'}},
                        {'body': 'What an accomplishment!',
                         'id': 30,
                         'postId': 23,
                         'user': {'id': 36, 'username': 'dalmondz'}}],
           'limit': 30,
           'skip': 0,
           'total': 340}

Process finished with exit code 0
"""
```

### Least Recently Used

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

#### limited

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

#### unbounded cache
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

## requests-cache

- [request cache library docs](https://requests-cache.readthedocs.io/en/stable/)

takes one minute
```python
import requests

session = requests.Session()
for i in range(60):
    session.get('https://httpbin.org/delay/1')
```

takes one second

```python
import requests_cache

session = requests_cache.CachedSession('demo_cache')
for i in range(60):
    session.get('https://httpbin.org/delay/1')
```

Define exactly how long to keep responses:

```python
from requests_cache import CachedSession
from datetime import timedelta

# Keep responses for 360 seconds
session = CachedSession('demo_cache', expire_after=360)

# Or use timedelta objects to specify other units of time
session = CachedSession('demo_cache', expire_after=timedelta(hours=1))
```

Use Cache-Control headers:

Use the cache_control parameter to enable automatic expiration based on Cache-Control and other standard HTTP headers sent by the server:

```python
from requests_cache import CachedSession

session = CachedSession('demo_cache', cache_control=True)
```

Settings:

```python
from datetime import timedelta
from requests_cache import CachedSession

session = CachedSession(
    'demo_cache',
    use_cache_dir=True,                # Save files in the default user cache dir
    cache_control=True,                # Use Cache-Control response headers for expiration, if available
    expire_after=timedelta(days=1),    # Otherwise expire responses after one day
    allowable_codes=[200, 400],        # Cache 400 responses as a solemn reminder of your failures
    allowable_methods=['GET', 'POST'], # Cache whatever HTTP methods you want
    ignored_parameters=['api_key'],    # Don't match this request param, and redact if from the cache
    match_headers=['Accept-Language'], # Cache a different response per language
    stale_if_error=True,               # In case of request errors, use stale cache data if possible
)
```

## profiling

### built-in profile

- [profile docs](https://docs.python.org/3/library/profile.html)

module profilehooks provides a simple decorator for our functions

```bash
pip install profilehooks
```
```python
from profilehooks import profile

# stdout=False -> don't print anything in the terminal
# filname -> path to the output file with profiling results
@profile(stdout=False, filename='baseline.prof')
def baseline():
    ...
```

### profile visualizer

#### snakeviz graphs

```bash
pip install snakeviz
```

#### gprof2dot flowchart

- [gh](https://github.com/jrfonseca/gprof2dot)

shows % of total execution time spent inside function

```bash
pip install gprof2dot
python -m gprof2dot -f pstats <profiling-results-file> | dot -Tpng -o output.png
```


#### threading aware

eg for non-Python libs like numpy, pytorch, scipy

- [yappi](https://github.com/sumerc/yappi) Yet Another Python Profiler, but this time multithreading, asyncio and gevent aware
- [viztracer](https://github.com/gaogaotiantian/viztracer) VizTracer is a low-overhead logging/debugging/profiling tool that can trace and visualize your python code execution
The front-end UI is powered by Perfetto. Use "AWSD" to zoom/navigate

#### gpu

- [pytorch profiler](https://pytorch.org/tutorials/recipes/recipes/profiler_recipe.html)

```python
def compute_big_sum(tensor: torch.Tensor):
    a = tensor.sum()
    b = tensor.pow(2).sum()
    c = tensor.sqrt().sum()
    
    sum_all = a + b + c
    torch.cuda.synchronize()  # add this after all your GPU operations
                              # to ensure correct time measurements
    sum_all_cpu = sum_all.cpu()
    
    return sum_all_cpu
```
