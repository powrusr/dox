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
## interfaces

### ABC

- an ABC is a way to define a template or a blueprint for a class and to ensure that any derived class adheres to a specific interface
- define common functionality across multiple classes and provide a way to check that a class has implemented all the necessary methods before it is used
- this makes ABCs useful when you need to define a strict interface for a set of related classes, ensuring that they all share a common set of behaviors

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass
```
Shape is an ABC that defines two abstract methods: area() and perimeter(). Any class that inherits from Shape must implement these methods

### protocol

- a protocol is simply a set of methods and/or attributes that a class must implement in order to be considered compatible with the protocol
- used to define the expected behavior of objects in a specific context, without requiring the overhead of a full class hierarchy
- the focus is on what an object can do, rather than what it is

```python
from typing import Protocol

class Device(Protocol):
    def connect(self) -> None:
        ...

    def disconnect(self) -> None:
        ...

    def send_message(self, message_type: MessageType, data: str) -> None:
```
Any class that implements these methods can be considered compatible with the Device Protocol

### summary

- ABCs are useful when you need to define a strict interface for a set of related classes, ensuring that they all share a common set of behaviors
- Protocols are useful when you need to define a more flexible interface that can be implemented by any class that provides the necessary methods and attributes


### ABC to protocol example

#### ABC

device.py
```python
from abc import ABC, abstractmethod

from iot.message import MessageType


class Device(ABC):
    @abstractmethod
    def connect(self) -> None:
        pass

    @abstractmethod
    def disconnect(self) -> None:
        pass

    @abstractmethod
    def send_message(self, message_type: MessageType, data: str) -> None:
        pass

    @abstractmethod
    def status_update(self) -> str:
        pass
```

devices.py
```python
from iot.device import Device
from iot.message import MessageType


class HueLight(Device):
    def connect(self) -> None:
        print("Connecting Hue light.")

    def disconnect(self) -> None:
        print("Disconnecting Hue light.")

    def send_message(self, message_type: MessageType, data: str) -> None:
        print(
            f"Hue light handling message of type {message_type.name} with data [{data}]."
        )

    def status_update(self) -> str:
        return "hue_light_status_ok"


class SmartSpeaker(Device):
    def connect(self) -> None:
        print("Connecting to Smart Speaker.")

    def disconnect(self) -> None:
        print("Disconnecting Smart Speaker.")

    def send_message(self, message_type: MessageType, data: str) -> None:
        print(
            f"Smart Speaker handling message of type {message_type.name} with data [{data}]."
        )

    def status_update(self) -> str:
        return "smart_speaker_status_ok"


class Curtains(Device):
    def connect(self) -> None:
        print("Connecting to Curtains.")

    def disconnect(self) -> None:
        print("Disconnecting Curtains.")

    def send_message(self, message_type: MessageType, data: str) -> None:
        print(
            f"Curtains handling message of type {message_type.name} with data [{data}]."
        )

    def status_update(self) -> str:
        return "curtains_status_ok"
```

diagnostics.py
```python
from iot.device import Device


def collect_diagnostics(device: Device) -> None:
    print("Connecting to diagnostics server.")
    status = device.status_update()
    print(f"Sending status update [{status}] to server.")
```

message.py
```python
from dataclasses import dataclass
from enum import Enum, auto


class MessageType(Enum):
    SWITCH_ON = auto()
    SWITCH_OFF = auto()
    CHANGE_COLOR = auto()
    PLAY_SONG = auto()
    OPEN = auto()
    CLOSE = auto()


@dataclass
class Message:
    device_id: str
    msg_type: MessageType
    data: str = ""
```

service.py
```python
import random
import string

from iot.device import Device
from iot.message import Message


def generate_id(length: int = 8):
    return "".join(random.choices(string.ascii_uppercase, k=length))


class IOTService:
    def __init__(self):
        self.devices: dict[str, Device] = {}

    def register_device(self, device: Device) -> str:
        device.connect()
        device_id = generate_id()
        self.devices[device_id] = device
        return device_id

    def unregister_device(self, device_id: str) -> None:
        self.devices[device_id].disconnect()
        del self.devices[device_id]

    def get_device(self, device_id: str) -> Device:
        return self.devices[device_id]

    def run_program(self, program: list[Message]) -> None:
        print("=====RUNNING PROGRAM======")
        for msg in program:
            self.devices[msg.device_id].send_message(msg.msg_type, msg.data)
        print("=====END OF PROGRAM======")
```

main.py
```python
from iot.devices import Curtains, HueLight, SmartSpeaker
from iot.diagnostics import collect_diagnostics
from iot.message import Message, MessageType
from iot.service import IOTService


def main() -> None:
    # create a IOT service
    service = IOTService()

    # create and register a few devices
    hue_light = HueLight()
    speaker = SmartSpeaker()
    curtains = Curtains()
    hue_light_id = service.register_device(hue_light)
    speaker_id = service.register_device(speaker)
    curtains_id = service.register_device(curtains)

    # create a few programs
    wake_up_program = [
        Message(hue_light_id, MessageType.SWITCH_ON),
        Message(speaker_id, MessageType.SWITCH_ON),
        Message(speaker_id, MessageType.PLAY_SONG, "Miles Davis - Kind of Blue"),
        Message(curtains_id, MessageType.OPEN),
    ]

    sleep_program = [
        Message(hue_light_id, MessageType.SWITCH_OFF),
        Message(speaker_id, MessageType.SWITCH_OFF),
        Message(curtains_id, MessageType.CLOSE),
    ]

    # run the programs
    service.run_program(wake_up_program)
    service.run_program(sleep_program)

    # collect some diagnostics
    collect_diagnostics(hue_light)


if __name__ == "__main__":
    main()
```

#### protocol

devices.py
```python
from iot.message import MessageType


class HueLightDevice:
    def connect(self) -> None:
        print("Connecting Hue light.")

    def disconnect(self) -> None:
        print("Disconnecting Hue light.")

    def send_message(self, message_type: MessageType, data: str) -> None:
        print(
            f"Hue light handling message of type {message_type.name} with data [{data}]."
        )

    def status_update(self) -> str:
        return "hue_light_status_ok"


class SmartSpeakerDevice:
    def connect(self) -> None:
        print("Connecting to Smart Speaker.")

    def disconnect(self) -> None:
        print("Disconnecting Smart Speaker.")

    def send_message(self, message_type: MessageType, data: str) -> None:
        print(
            f"Smart Speaker handling message of type {message_type.name} with data [{data}]."
        )

    def status_update(self) -> str:
        return "smart_speaker_status_ok"


class CurtainsDevice:
    def connect(self) -> None:
        print("Connecting to Curtains.")

    def disconnect(self) -> None:
        print("Disconnecting Curtains.")

    def send_message(self, message_type: MessageType, data: str) -> None:
        print(
            f"Curtains handling message of type {message_type.name} with data [{data}]."
        )

    def status_update(self) -> str:
        return "curtains_status_ok"
```

diagnostics.py
```python
from typing import Protocol


class DiagnosticsSource(Protocol):
    def status_update(self) -> str:
        ...


def collect_diagnostics(device: DiagnosticsSource) -> None:
    print("Connecting to diagnostics server.")
    status = device.status_update()
    print(f"Sending status update [{status}] to server.")
```

message.py
```python
from dataclasses import dataclass
from enum import Enum, auto


class MessageType(Enum):
    SWITCH_ON = auto()
    SWITCH_OFF = auto()
    CHANGE_COLOR = auto()
    PLAY_SONG = auto()
    OPEN = auto()
    CLOSE = auto()


@dataclass
class Message:
    device_id: str
    msg_type: MessageType
    data: str = ""
```

service.py
```python
import random
import string
from typing import Protocol

from iot.message import Message, MessageType


def generate_id(length: int = 8):
    return "".join(random.choices(string.ascii_uppercase, k=length))


class Device(Protocol):
    def connect(self) -> None:
        ...

    def disconnect(self) -> None:
        ...

    def send_message(self, message_type: MessageType, data: str) -> None:
        ...


class IOTService:
    def __init__(self):
        self.devices: dict[str, Device] = {}

    def register_device(self, device: Device) -> str:
        device.connect()
        device_id = generate_id()
        self.devices[device_id] = device
        return device_id

    def unregister_device(self, device_id: str) -> None:
        self.devices[device_id].disconnect()
        del self.devices[device_id]

    def get_device(self, device_id: str) -> Device:
        return self.devices[device_id]

    def run_program(self, program: list[Message]) -> None:
        print("=====RUNNING PROGRAM======")
        for msg in program:
            self.devices[msg.device_id].send_message(msg.msg_type, msg.data)
        print("=====END OF PROGRAM======")
```

main.py
```python
from iot.devices import CurtainsDevice, HueLightDevice, SmartSpeakerDevice
from iot.diagnostics import collect_diagnostics
from iot.message import Message, MessageType
from iot.service import IOTService


class TestingDevice:
    def status_update(self) -> str:
        return "test_status_ok"


def main() -> None:
    # create a IOT service
    service = IOTService()

    # create and register a few devices
    hue_light = HueLightDevice()
    speaker = SmartSpeakerDevice()
    curtains = CurtainsDevice()
    hue_light_id = service.register_device(hue_light)
    speaker_id = service.register_device(speaker)
    curtains_id = service.register_device(curtains)

    # create a few programs
    wake_up_program = [
        Message(hue_light_id, MessageType.SWITCH_ON),
        Message(speaker_id, MessageType.SWITCH_ON),
        Message(speaker_id, MessageType.PLAY_SONG, "Miles Davis - Kind of Blue"),
        Message(curtains_id, MessageType.OPEN),
    ]

    sleep_program = [
        Message(hue_light_id, MessageType.SWITCH_OFF),
        Message(speaker_id, MessageType.SWITCH_OFF),
        Message(curtains_id, MessageType.CLOSE),
    ]

    # run the programs
    service.run_program(wake_up_program)
    service.run_program(sleep_program)

    # collect some diagnostics
    testing_device = TestingDevice()
    collect_diagnostics(testing_device)


if __name__ == "__main__":
    main()
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


