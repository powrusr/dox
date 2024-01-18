# design patterns

## guard clauses

bad

```python
# Sorry for the silly example
def is_platypus(self):
    if self.is_mammal():
        if self.has_fur():
            if self.has_beak():
                if self.has_tail():
                    if self.can_swim():
                        # It's a platypus!
                        return True
    # Not a platypus
    return False
```
good

```python
def is_platypus(self):
    # Not a platypus for everything below
    if not self.is_mammal():
        return False
    if not self.has_fur():
        return False
    if not self.has_beak():
        return False
    if not self.has_tail():
        return False
    if not self.can_swim():
        return False
    # Finally, it's a platypus!
    return True
```

```python
def func_not_guarded(self, param):
    if param == 'something':
        self.counter += 1
        if self.counter > 10:
            self.reached_ten()
        else:
            if self.counter < 5:
                self.has_not_reached_5()
            else:
                self.has_not_reached_5()
    else:
        self.counter -= 1


def func_guarded(self, param):
    if param != 'something':
        self.counter -= 1
        return
    self.counter += 1
    if self.counter > 10:
        self.reached_ten()
        return
    if self.counter < 5:
        self.has_not_reached_5()
        return
    self.has_not_reached_5()
```

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


