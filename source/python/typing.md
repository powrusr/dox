# typing

[cheatsheet](https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html)

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


