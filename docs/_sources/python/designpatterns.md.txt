# design patterns

## factory

allow user input to translate to objects being created
by coupling the string to the class

select key which has class, call that with extra parentheses to initialize
a class instance

```python
dictionary["user_input"](*args,**kwargs) 
```

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius
    # Class implementation...

class Square:
    def __init__(self, side):
        self.side = side
    # Class implementation...

def shape_factory(shape_name, *args, **kwargs):
    shapes = {"circle": Circle, "square": Square}
    return shapes[shape_name](*args, **kwargs)
```


