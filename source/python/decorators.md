# decorators

add logic to function without changing it

## timer

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

## backoff

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
