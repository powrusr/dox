# decorators

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

