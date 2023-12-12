# collections

## definition

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

## namedtuple

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

## chainmap

   ```python
   >>> baseline = {'music': 'bach', 'art': 'rembrandt'}
   >>> adjustments = {'art': 'van gogh', 'opera': 'carmen'}
   >>> list(ChainMap(adjustments, baseline))
   ['music', 'art', 'opera']
   ```
