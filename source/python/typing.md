# typing

[official docs](https://docs.python.org/3/library/typing.html)
[community docs](https://typing.readthedocs.io/en/latest/)
[cheatsheet](https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html)
## type aliases

```python
from collections.abc import Sequence

type ConnectionOptions = dict[str, str]
type Address = tuple[str, int]
type Server = tuple[Address, ConnectionOptions]

def broadcast_message(message: str, servers: Sequence[Server]) -> None:
    ...

# The static type checker will treat the previous type signature as
# being exactly equivalent to this one.
def broadcast_message(
        message: str,
        servers: Sequence[tuple[tuple[str, int], dict[str, str]]]) -> None:
    ...
```

The type statement is new in Python 3.12. For backwards compatibility, type aliases can also be created through simple assignment:

```python
Vector = list[float]
Or marked with TypeAlias to make it explicit that this is a type alias, not a normal variable assignment:
```

```python
from typing import TypeAlias

Vector: TypeAlias = list[float]
```

## NewType

Use the NewType helper to create distinct types:

```python
from typing import NewType

UserId = NewType('UserId', int)
some_id = UserId(524313)
```

The static type checker will treat the new type as if it were a subclass of the original type. This is useful in helping catch logical errors:

```python
def get_user_name(user_id: UserId) -> str:
    ...

# passes type checking
user_a = get_user_name(UserId(42351))

# fails type checking; an int is not a UserId
user_b = get_user_name(-1)
```

You may still perform all int operations on a variable of type UserId, but the result will always be of type int. This lets you pass in a UserId wherever an int might be expected, but will prevent you from accidentally creating a UserId in an invalid way:

```python
# 'output' is of type 'int', not 'UserId'
output = UserId(23413) + UserId(54341)
```

## annotating callables

Callable[[int], str] signifies a function that takes a single parameter of type int and returns a str.

```python
from collections.abc import Callable, Awaitable

def feeder(get_next_item: Callable[[], str]) -> None:
    ...  # Body

def async_query(on_success: Callable[[int], None],
                on_error: Callable[[int, Exception], None]) -> None:
    ...  # Body

async def on_update(value: str) -> None:
    ...  # Body

callback: Callable[[str], Awaitable[None]] = on_update
```

### ellepsis ...

... means any parameter list is acceptable

```python
def concat(x: str, y: str) -> str:
    return x + y

x: Callable[..., str]
x = str     # OK
x = concat  # Also OK
```
## ABCs

Used in typing so good to have an overview
[abc docs](https://docs.python.org/3/library/collections.abc.html)

table of collections abstract base classes

|ABC|Inherits from|Abstract Methods|Mixin Methods|
|--|--|--|--|
|Container ||__contains__||
|Hashable ||__hash__||
|Iterable ||__iter__||	 
|Iterator |Iterable|__next__|__iter__|
|Reversible |Iterable|	__reversed__	 
|Generator 	|Iterator|send, throw|close, __iter__, __next__|
|Sized |	 	__len__	 
|Callable |	 	__call__	 
|Collection 	|Sized,<br> |Iterable,<br>|Container|	__contains__, __iter__, __len__	 
|Sequence	|Reversible,<br> |Collection<br>|	__getitem__, __len__	__contains__, __iter__, __reversed__, index, and count
|MutableSequence	|Sequence	__getitem__, __setitem__, __delitem__, __len__, insert	Inherited |Sequence methods and append, reverse, extend, pop, remove, and __iadd__
|ByteString	|Sequence	__getitem__, __len__	Inherited |Sequence methods
|Set	|Collection	__contains__, __iter__, __len__	__le__, __lt__, __eq__, __ne__, __gt__, __ge__, __and__, __or__, __sub__, __xor__, and isdisjoint
|MutableSet	|Set	__contains__, __iter__, __len__, add, discard	Inherited |Set methods and clear, pop, remove, __ior__, __iand__, __ixor__, and __isub__
|`Mapping`	|`Collection`	__getitem__, __iter__, __len__	__contains__, keys, items, values, get, __eq__, and __ne__
|`MutableMapping`	|`Mapping`	__getitem__, __setitem__, __delitem__, __iter__, __len__	Inherited |`Mapping` methods and pop, popitem, clear, update, and setdefault
|`MappingView`	|`Sized`	 	__len__
|`ItemsView`	|`MappingView`, |`Set`	 	__contains__, __iter__
|`KeysView`	|`MappingView`, |`Set`	 	__contains__, __iter__
|`ValuesView`	|`MappingView`, |`Collection`	 	__contains__, __iter__
|`Awaitable` |	 	__await__	 
|`Coroutine` 	|`Awaitable`	send, throw	close
|`AsyncIterable` |	 	__aiter__	 
|`AsyncIterator` 	|`AsyncIterable`	__anext__	__aiter__
|`AsyncGenerator` 	|`AsyncIterator`	asend, athrow	aclose, __aiter__, __anext__
|`Buffer` |	 	__buffer__	 

## generics

```python
from collections.abc import Mapping, Sequence

class Employee: ...

# Sequence[Employee] indicates that all elements in the sequence
# must be instances of "Employee".
# Mapping[str, str] indicates that all keys and all values in the mapping
# must be strings.
def notify_by_email(employees: Sequence[Employee],
                    overrides: Mapping[str, str]) -> None: ...
```
Generic functions and classes can be parameterized by using type parameter syntax:

```python
from collections.abc import Sequence

def first[T](l: Sequence[T]) -> T:  # Function is generic over the TypeVar "T"
    return l[0]
```
Or by using the TypeVar factory directly:

```python
from collections.abc import Sequence
from typing import TypeVar

U = TypeVar('U')                  # Declare type variable "U"

def second(l: Sequence[U]) -> U:  # Function is generic over the TypeVar "U"
    return l[1]
```

## override

Allows our syntax checker to detect when a base class was altered but child classes using it aren't aware of the change

```python
import random
from dataclasses import dataclass
from typing import Self, override


def generate_account_number() -> str:
    """Generate a random eleven-digit account number"""
    account_number = str(random.randrange(10_000_000_000, 100_000_000_000))
    return f"{account_number[:4]}.{account_number[4:6]}.{account_number[6:]}"


@dataclass
class BankAccount:
    account_number: str
    balance: float

    @classmethod
    def from_balance(cls, balance: float) -> Self:
        return cls(generate_account_number(), balance)

    def deposit(self, amount: float) -> None:
        self.balance += amount

    def withdraw(self, amount: float) -> None:
        self.balance -= amount


@dataclass
class SavingsAccount(BankAccount):
    interest: float

    def add_interest(self) -> None:
        self.balance *= 1 + self.interest / 100

    @classmethod
    @override  # track parent class method for changes
    def from_balance(cls, balance: float, interest: float = 1.0) -> Self:
        return cls(generate_account_number(), balance, interest)

    @override
    def withdraw(self, amount: float) -> None:
        self.balance -= int(amount) if amount > 100 else amount
```

## TypedDict & **kwargs

```python
>>> from typing import TypedDict
>>> class Options(TypedDict):
...     line_width: int
...     level: str
...     propagate: bool
```
to use TypedDict on `**kwargs` you need to wrap it inside **Unpack**

```python
>>> from typing import Unpack
>>> def show_options(program_name: str, **kwargs: Unpack[Options]) -> None:
...     print(program_name.upper())
...     for option, value in kwargs.items():
...         print(f"{option:<15} {value}")
```
You use Unpack to say that the typed dictionary provides different type hints for different keyword arguments. If you use \*\*kwargs: Options, then you’re saying that each keyword argument should be an Options dictionary.

By default, all fields in a typed dictionary are required.
That means that you must pass in line_width, level, and propagate and can’t leave any of them out.
That’s not usually what you intend when using \*\*kwargs.
You can switch this so that none of the keys are required by specifying **total=False** when you define the typed dictionary:

```python
>>> from typing import TypedDict
>>> class Options(TypedDict, total=False):
...     line_width: int
...     level: str
...     propagate: bool
```

In this case, all the arguments are optional.
For more fine-grain control, you can also use [Required](https://docs.python.org/3/library/typing.html#typing.Required) and [NotRequired](https://docs.python.org/3/library/typing.html#typing.NotRequired). For example, to denote that only level is required, you can do the following

```python
>>> from typing import Required, TypedDict
>>> class Options(TypedDict, total=False):
...     line_width: int
...     level: Required[str]
...     propagate: bool
```

```python
from typing import Required, TypedDict, Unpack


class Options(TypedDict, total=False):
    line_width: int
    level: Required[str]
    propagate: bool


def show_options(program_name: str, **kwargs: Unpack[Options]) -> None:
    print(program_name.upper())
    for option, value in kwargs.items():
        print(f"{option:<15} {value}")


def show_options_explicit(
    program_name: str,
    *,
    level: str,
    line_width: int | None = None,
    propagate: bool | None = None,
) -> None:
    options = {
        "line_width": line_width,
        "level": level,
        "propagate": propagate,
    }
    print(program_name.upper())
    for option, value in options.items():
        if value is not None:
            print(f"{option:<15} {value}")


show_options("logger", line_width=80, level="INFO", propagate=False)
```

# annotated typing

[annotated-types](https://github.com/annotated-types/annotated-types)


# pedantic

use pedantic for extra validation & (de)serialization power

[pydantic docs](https://docs.pydantic.dev/latest/)

## types

[more on types on pydantic site](https://docs.pydantic.dev/latest/concepts/types/)

### annotated

```python
# or `from typing import Annotated` for Python 3.9+
from typing_extensions import Annotated

from pydantic import Field, TypeAdapter, ValidationError

PositiveInt = Annotated[int, Field(gt=0)]

ta = TypeAdapter(PositiveInt)

print(ta.validate_python(1))
#> 1

try:
    ta.validate_python(-1)
except ValidationError as exc:
    print(exc)
    """
    1 validation error for constrained-int
      Input should be greater than 0 [type=greater_than, input_value=-1, input_type=int]
    """


```


## validation

```python
from datetime import datetime

from pydantic import BaseModel, PositiveInt


class User(BaseModel):
    id: int  
    name: str = 'John Doe'  
    signup_ts: datetime | None  
    tastes: dict[str, PositiveInt]  


external_data = {
    'id': 123,
    'signup_ts': '2019-06-01 12:22',  
    'tastes': {
        'wine': 9,
        b'cheese': 7,  
        'cabbage': '1',  
    },
}

user = User(**external_data)  

print(user.id)  
#> 123
print(user.model_dump())  
"""
{
    'id': 123,
    'name': 'John Doe',
    'signup_ts': datetime.datetime(2019, 6, 1, 12, 22),
    'tastes': {'wine': 9, 'cheese': 7, 'cabbage': 1},
}
"""
```

If validation fails, Pydantic will raise an error with a breakdown of what was wrong:

```python
# continuing the above example...

from pydantic import ValidationError


class User(BaseModel):
    id: int
    name: str = 'John Doe'
    signup_ts: datetime | None
    tastes: dict[str, PositiveInt]


external_data = {'id': 'not an int', 'tastes': {}}  

try:
    User(**external_data)  
except ValidationError as e:
    print(e.errors())
    """
    [
        {
            'type': 'int_parsing',
            'loc': ('id',),
            'msg': 'Input should be a valid integer, unable to parse string as an integer',
            'input': 'not an int',
            'url': 'https://errors.pydantic.dev/2/v/int_parsing',
        },
        {
            'type': 'missing',
            'loc': ('signup_ts',),
            'msg': 'Field required',
            'input': {'id': 'not an int', 'tastes': {}},
            'url': 'https://errors.pydantic.dev/2/v/missing',
        },
    ]
    """
```


