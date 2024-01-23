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
|Container ||`__contains__`||
|Hashable ||`__hash__`||
|Iterable ||`__iter__`||	 
|Iterator |Iterable|`__next__`|`__iter__`|
|Reversible |Iterable|`__reversed__`|	 
|Generator 	|Iterator|send, throw|close, `__iter__`, `__next__`|
|Sized ||`__len__`||
|Callable ||`__call__`||
|Collection	|Sized, Iterable,<br>Container|`__contains__`,<br>`__iter__`,<br>`__len__`||	 
|Sequence	|Reversible,<br> Collection<br>|	`__getitem__`,<br>`__len__`|`__contains__`, `__iter__`, `__reversed__`, `index`, and `count`|
|MutableSequence	|Sequence|`__getitem__`,<br>`__setitem__`,<br>`__delitem__`,<br>`__len__`,<br>`insert`|Inherited Sequence methods and append, reverse, extend, pop, remove, and `__iadd__`|
|ByteString	|Sequence|`__getitem__`,<br>`__len__`|Inherited Sequence methods|
|Set	|Collection|`__contains__`,<br> `__iter__`,<br> `__len__`<br>|	`__le__`, `__lt__`, `__eq__`, `__ne__`, `__gt__`, `__ge__`, `__and__`, `__or__`, `__sub__`, `__xor__`, and `isdisjoint`|
|MutableSet	|Set|`__contains__`, `__iter__`, `__len__`, add, discard	Inherited |Set methods and `clear`, `pop`, `remove`, `__ior__`, `__iand__`, `__ixor__`, and `__isub__`
|Mapping	|Collection|`__getitem__`, `__iter__`, `__len__`|`__contains__`, `keys`, `items`, `values`, `get`, `__eq__`, and `__ne__`
|MutableMapping	|Mapping|`__getitem__`, `__setitem__`, `__delitem__`, `__iter__`, `__len__`|Inherited Mapping methods and `pop`, `popitem`, `clear`, `update`, and `setdefault`|
|MappingView	|Sized|	 	|`__len__`|
|ItemsView	|MappingView, Set|	 	|`__contains__`, `__iter__`|
|KeysView	|MappingView, Set|	 	|`__contains__`, `__iter__`|
|ValuesView	|MappingView, Collection|	 	|`__contains__`, `__iter__`|
|Awaitable |	 	|`__await__`	 ||
|Coroutine 	|Awaitable|	`send`, `throw`|`close`|
|AsyncIterable |	 |	`__aiter__`||	 
|AsyncIterator 	|AsyncIterable|	`__anext__`	|`__aiter__`|
|AsyncGenerator 	|AsyncIterator|	`asend`, `athrow`|`aclose`, `__aiter__`, `__anext__`|
|`Buffer` |	 	|`__buffer__`	 ||

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

## deserialize

### from dict
```python
data = {
    "first_name": "John",
    "last_name": "Smith",
    "age": 42
}

p = Person.model_validate(data)
p
Person(first_name='John', last_name='Smith', age=42)
```
### from json

```python
data_json = '''
{
    "first_name": "John",
    "last_name": "Smith",
    "age": 42
}
'''

p = Person.model_validate_json(data_json)
p
Person(first_name='John', last_name='Smith', age=42)
```
We can inspect the model's fields this way, to see how they are currently defined:

```python
Person.model_fields
{'first_name': FieldInfo(annotation=str, required=True),
 'last_name': FieldInfo(annotation=str, required=True),
 'age': FieldInfo(annotation=int, required=True)}
```
## serialize

```python
p.model_dump()
{'id_': 100, 'first_name': 'John', 'last_name': 'Smith', 'age': 42}
```
```python
p.model_dump_json()
'{"id_":100,"first_name":"John","last_name":"Smith","age":42}'
```
As you can see, serialization uses the field names, not the aliases to serialize.

Since we have aliases, we could, if we wanted to, also serialize using the aliases instead of the field names:

```python
p.model_dump(by_alias=True)
{'id': 100, 'First Name': 'John', 'LASTNAME': 'Smith', 'age in years': 42}
```
```python
p.model_dump_json(by_alias=True)
'{"id":100,"First Name":"John","LASTNAME":"Smith","age in years":42}'
```

### custom serializer

eg:
- specify how many digits of a float after decimal point
- specify how datetime objects are serialized from json

default:
```python
class Model(BaseModel):
    number: float

m = Model(number=1.0)
m.model_dump()
{'number': 1.0}

m = Model(number=1/3)
m.model_dump()
{'number': 0.3333333333333333}
```

custom:
careful since we actually have tow modes of serialization: to a Python dictionary (so Python objects), and to JSON (so strings).
to customize the float serialization to rounding to 2 decimal places, in both dictionary and JSON serialization

```python
from pydantic import field_serializer

class Model(BaseModel):
    number: float

    @field_serializer("number")
    def serialize_float(self, value):
        return round(value, 2)


m = Model(number=1/3)
m.model_dump()
{'number': 0.33}

m.model_dump_json()
'{"number":0.33}'
```

datetimes get serialized to JSON using the isoformat() method of datetimes:

default:
```python
dt = datetime.now(timezone.utc)
dt.isoformat()
'2023-12-06T14:33:05.667170+00:00'

class Model(BaseModel):
    dt: datetime

m = Model(dt=datetime.now(timezone.utc))
m.model_dump_json()
'{"dt":"2023-12-06T14:33:05.672674Z"}'
```
custom:
```python
class Model(BaseModel):
    number: float
    dt: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))

    @field_serializer("number")
    def serialize_float(self, value):
        return round(value, 2)

    @field_serializer("dt", when_used="json-unless-none")
    def serialize_datetime_to_json(self, value):
        return value.strftime("%Y/%-m/%-d %I:%M %p")

        
m = Model(number=1/3)
m
Model(number=0.3333333333333333, dt=datetime.datetime(2023, 12, 6, 14, 33, 5, 692861, tzinfo=datetime.timezone.utc))

m.model_dump()
{'number': 0.33,
 'dt': datetime.datetime(2023, 12, 6, 14, 33, 5, 692861, tzinfo=datetime.timezone.utc)}

m.model_dump_json()
'{"number":0.33,"dt":"2023/12/6 02:33 PM"}'
```
dt serialization remained unaffacted, but when we serialize to JSON, our custom serializer is used

## field

### alias

Sometimes the data we are attempting to deserialize uses names that we simply do not, or cannot use in our model.

For example consider this data we would like to model using Pydantic:

```python
data = {
    "id": 100,
    "First Name": "John",
    "LASTNAME": "Smith",
    "age in years": 42,
}
```

Pydantic has a way to define an alternative name to our field names, called aliases.
Note the spaces

```python
from pydantic import Field


class Person(BaseModel):
    id_: int = Field(alias="id")
    first_name: str = Field(alias="First Name")
    last_name: str = Field(alias="LASTNAME")
    age: int = Field(alias="age in years")
p = Person.model_validate(data)
p
Person(id_=100, first_name='John', last_name='Smith', age=42)
```
### allow not using alias

```python
from pydantic import ConfigDict

class Person(BaseModel):
    model_config = ConfigDict(populate_by_name=True)
    
    first_name: str | None = Field(alias="firstName", default=None)
    last_name: str = Field(alias="lastName")
```
And now we can deserialize using either the alias or the field name:
```python
p = Person(first_name="John", lastName="Smith")
p
Person(first_name='John', last_name='Smith')

data = {
    "first_name": "John",
    "lastName": "Smith"
}

p = Person.model_validate(data)
p
Person(first_name='John', last_name='Smith')
```

### defaults

When we used the Field object to define an alias, we lost the ability to set our fields to some default value.

```python
class Person(BaseModel):
    first_name: str | None = Field(alias="firstName", default=None)
    last_name: str = Field(alias="lastName")
data = {
    "lastName": "Smith"
}

p = Person.model_validate(data)
p
Person(first_name=None, last_name='Smith')
```

#### mutable defaults

Pydantic can handle is setting default values to mutable objects  
something that is usually problematic in Python, and disallowed (by default) in dataclasses

defining defaults this way is aok in Pydantic (Pydantic basically identifies mutable defaults, and creates a deepcopy of the default when creating new instances of the model):

```python
class Model(BaseModel):
    numbers: list[int] = []

m1 = Model()
m2 = Model()

m1.numbers.extend([1, 2, 3])
m1
Model(numbers=[1, 2, 3])

m2
Model(numbers=[])
```

#### default factory

to generate a default not as a static value, but rather as a value that should be calculated each time an instance is created that needs the default.

We can do that using a default factory. Basically, we provide a function that Pydantic will call to generate the default value each time a model instance is created that requires that default.

```python
from datetime import datetime, timezone

class Log(BaseModel):
    dt: datetime = Field(default_factory=lambda: datetime.now(timezone.utc))
    message: str

log1 = Log(message="message 1")
log2 = Log(message="message 2")

log1
Log(dt=datetime.datetime(2023, 12, 6, 14, 33, 5, 648812, tzinfo=datetime.timezone.utc), message='message 1')

log2
Log(dt=datetime.datetime(2023, 12, 6, 14, 33, 5, 650873, tzinfo=datetime.timezone.utc), message='message 2')
```

## validation

Validators are not just validation functions, they are also transformation functions  
Pydantic's validators can modify the type of the data being deserialized to coerce it into the proper type

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

### custom validators

- before validator  
run before Pydantic has a chance to validate and coerce the data according to our field definition

> can be very handy to provide custom parsing of data that Pydantic would otherwise be unable to do. For example deserializing a date provided in a format that Python does not recognize (e.g. 2024/1/1 3:15pm)


- after validator  
after Pydantic has already processed the raw data, validated it and coerced it to the proper type, as defined by the field definition

```python
from pydantic import field_validator

class Model(BaseModel):
    absolute: int

    @field_validator("absolute")
    @classmethod
    def make_absolute(cls, value):
        return abs(value)


Model(absolute=-10)
Model(absolute=10)
```

NOTE that our custom validator, being an after validator, will get called once Pydantic tried to parse the input data to an int. If that validation fails, our custom validator will not even get called.

```python
class Model(BaseModel):
    absolute: int

    @field_validator("absolute")
    @classmethod
    def make_absolute(cls, value):
        print(f"running custom validator: {value=}, {type(value)=}")
        return abs(value)


Model(absolute=-10)
running custom validator: value=-10, type(value)=<class 'int'>

Model(absolute=10)
```

Let's pass something that is not an integer, but can be coerced to an integer:

```python
Model(absolute="-10")
running custom validator: value=-10, type(value)=<class 'int'>
Model(absolute=10)
```
As you can see, our validator received an integer, not a string

And if Pydantic's validation fails, our validator is not called:

```python
try:
    Model(absolute="abc")
except ValidationError as ex:
    print(ex)
```
```
1 validation error for Model
absolute
  Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='abc', input_type=str]
    For further information visit https://errors.pydantic.dev/2.4/v/int_parsing
```

Notice our custom validator's print statement never executed.
We can of course, also use custom validators to perform validation.

Let's say we want to define a field that should be a list of unique integers.

```python
class Model(BaseModel):
    numbers: list[int] = []

    @field_validator("numbers")
    @classmethod
    def ensure_unique(cls, numbers):
        if len(set(numbers)) != len(numbers):
            raise ValueError("elements must be unique")
        return numbers

Model(numbers=[1, 2, 3])
```
```python
try:
    Model(numbers=[1, 1, 2, 3])
except ValidationError as ex:
    print(ex)
```
```
1 validation error for Model
numbers
  Value error, elements must be unique [type=value_error, input_value=[1, 1, 2, 3], input_type=list]
    For further information visit https://errors.pydantic.dev/2.4/v/value_error
```
Notice how I raised a ValueError error.  
 If you want to raise a Pydantic ValidationException, you should raise a ValueError  
 Most of the other exception types (such as TypeError, KeyError, etc) will bubble up as those exceptions, not a ValidationError exception)  
 There are a few other errors you can raise that will result in a ValidationError exception, but by far ValueError is the easiest and safest way to do so

## nested models

```python
data = {
    "firstName": "Arthur",
    "lastName": "Bourne",
    "born": {
        "place": {
            "country": "Mars Colony",
            "city": "Central City",
        },
        "date": "3001-01-01",
    }
}
```
As you can see we have three levels of nested dictionaries - which we can model this way:

```python
from datetime import date

class Place(BaseModel):
    country: str
    city: str

class Born(BaseModel):
    place: Place
    dt: date = Field(alias="date")
    
class Person(BaseModel):
    first_name: str | None = Field(alias="firstName", default=None)
    last_name: str = Field(alias="lastName")
    born: Born


arthur = Person.model_validate(data)
arthur
Person(first_name='Arthur', last_name='Bourne', born=Born(place=Place(country='Mars Colony', city='Central City'), dt=datetime.date(3001, 1, 1)))

# access all these fields using object dot notation:
arthur.born.place.country
'Mars Colony'
```

