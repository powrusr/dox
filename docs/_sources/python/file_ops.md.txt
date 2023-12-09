# glob

https://docs.python.org/3/library/glob.html#module-glob

```python

"""
dir/1.gif
dir/2.txt
dir/card.gif
dir/sub/3.txt
"""

import glob
glob.glob('./[0-9].*')
['./1.gif', './2.txt']

glob.glob('*.gif')
['1.gif', 'card.gif']

glob.glob('?.gif')
['1.gif']

glob.glob('**/*.txt', recursive=True)
['2.txt', 'sub/3.txt']

glob.glob('./**/', recursive=True)
['./', './sub/']
```

If the directory contains files starting with . they won’t be matched by default
