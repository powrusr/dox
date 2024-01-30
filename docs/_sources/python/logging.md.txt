# logging

- don't use the root logger directly bruh
- use your own: `logger = logging.getLogger("__name__")`
- in a big app **create 1 logger per major component** (not for every file)
- use dictConfig! `logging.config.dictConfig(config)`
- use ISO-8601 timestamps and include the timezone! eg `"datefmt": "%Y-%m-%dT%H:%M:%S%z"`
- create custom json formatter, so you don't have to parse weird traceback messages in your text logs
- use extra `mylogger.debug("debug message", extra={"well": "hello there"}), it will be added to the logrecord
- logging calls are synchronous and blocking so use a [QueueHandler](https://docs.python.org/3/library/logging.handlers.html#queuehandler)/[QueueListener](https://docs.python.org/3/library/logging.handlers.html#queuelistener)
- don't **configure** logging on library code


## formatters

### custom

```json
"formatters": {
  "json": {
    "()": "mylogger.MyJSONFormatter",
    "fmt_keys": {
      "level": "levelname",
      "other_keys": "here",
      "thread_name": "threadName"
    }
  }
}
```
writing your own formatter class

```python
class MyJSONFormatter(logging.Formatter):

    def __init__(self, *, fmt_keys: dict[str, str] | None = None,):
	    super().__init__()
        self.fmt_keys = fmt_keys if fmt_keys is not None else {}
...    
```

## logrecord attributes

- [logrecord attributes docs](https://docs.python.org/3/library/logging.html#logrecord-attributes)

 **Attribute name** | **Format** | **Description** 
---|---|---
args | You shouldn’t need to format this yourself. | The tuple of arguments merged into msg to produce message, or a dict whose values are used for the merge (when there is only one argument, and it is a dictionary). 
asctime | `%(asctime)s` | Human-readable time when the [LogRecord](https://docs.python.org/3/library/logging.html#logging.LogRecord) was created.  By default this is of the form ‘2003-07-08 16:49:45,896’ (the numbers after the comma are millisecond portion of the time). 
created | `%(created)f` | Time when the LogRecord was created (as returned by [time.time()](https://docs.python.org/3/library/time.html#time.time)). 
exc_info | You shouldn’t need to format this yourself. | Exception tuple (à la `sys.exc_info`) or, if no exception has occurred, None. 
filename | `%(filename)s` | Filename portion of `pathname`. 
funcName | `%(funcName)s` | Name of function containing the logging call. 
levelname | `%(levelname)s` | Text logging level for the message (`'DEBUG'`, `'INFO'`, `'WARNING'`, `'ERROR'`, `'CRITICAL'`). 
levelno | `%(levelno)s` | Numeric logging level for the message (DEBUG, INFO, WARNING, ERROR, CRITICAL). 
lineno | `%(lineno)d` | Source line number where the logging call was issued (if available). 
message | `%(message)s` | The logged message, computed as `msg % args`. This is set when [Formatter.format()](https://docs.python.org/3/library/logging.html#logging.Formatter.format) is invoked. 
module | `%(module)s` | Module (name portion of `filename`).
msecs | `%(msecs)d` | Millisecond portion of the time when the LogRecord was created. 
msg | You shouldn’t need to format this yourself. | The format string passed in the original logging call. Merged with `args` to produce `message`, or an arbitrary object (see [Using arbitrary objects as messages](https://docs.python.org/3/howto/logging.html#arbitrary-object-messages)). 
name | `%(name)s` | Name of the logger used to log the call. 
pathname | `%(pathname)s` | Full pathname of the source file where the logging call was issued (if available). 
process | `%(process)d` | Process ID (if available). 
processName | `%(processName)s` | Process name (if available). 
relativeCreated | `%(relativeCreated)d` | Time in milliseconds when the LogRecord was created, relative to the time the logging module was loaded. 
stack_info | You shouldn’t need to format this yourself. | Stack frame information (where available) from the bottom of the stack in the current thread, up to and including the stack frame of the logging call which resulted in the creation of this record. 
thread | `%(thread)d` | Thread ID (if available). 
threadName | `%(threadName)s` | Thread name (if available). 
taskName | `%(taskName)s` | [asyncio.Task](https://docs.python.org/3/library/asyncio-task.html#asyncio.Task) name (if available). 



## levels
<div style="margin-left: 50;
            margin-right: auto;
            width: 30%">

Level| Numeric value
--|--
CRITICAL| 50
ERROR| 40
WARNING| 30
INFO| 20
DEBUG| 10
NOTSET| 0

</div>



## filter

logger filter: drop or alter log record
- drop records that begin with some regex pattern 
- eg sensor sensitive data

handler filter: specific to handler

> **__note__**: keep all filters on the logger to avoid complexity, but leave on propagation so that all messages bubble up to the root logger
> **__note__**: this also ensure log messages generated from 3rd parties are formatted like own app log 
messages

## handler

tells you to pass it on to eg
- stdout
- a file
- e-mail
- log service

like logger you can specify log level in **each handler**

if handler drops log record, it moves on to the next handler
if main logger drops log record, its gone

> **__note__**: same as with filter keep all handlers on your logger

### useful handlers

In addition to the base Handler class, many useful subclasses are provided:

1. [StreamHandler](https://docs.python.org/3/library/logging.handlers.html#logging.StreamHandler) instances send messages to streams (file-like objects).
2. [FileHandler](https://docs.python.org/3/library/logging.handlers.html#logging.FileHandler) instances send messages to disk files.
3. [BaseRotatingHandler](https://docs.python.org/4/library/logging.handlers.html#logging.handlers.BaseRotatingHandler) is the base class for handlers that rotate log files at a certain point. It is not meant to be instantiated directly. Instead, use RotatingFileHandler or TimedRotatingFileHandler.
4. [RotatingFileHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.RotatingFileHandler) instances send messages to disk files, with support for maximum log file sizes and log file rotation.
5. [TimedRotatingFileHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.TimedRotatingFileHandler) instances send messages to disk files, rotating the log file at certain timed intervals.
6. [SocketHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.SocketHandler) instances send messages to TCP/IP sockets. Since 3.4, Unix domain sockets are also supported.
7. [DatagramHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.DatagramHandler) instances send messages to UDP sockets. Since 3.4, Unix domain sockets are also supported.
8. [SMTPHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.SMTPHandler) instances send messages to a designated email address.
9. [SysLogHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.SysLogHandler) instances send messages to a Unix syslog daemon, possibly on a remote machine.
10. [NTEventLogHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.NTEventLogHandler) instances send messages to a Windows NT/2000/XP event log.
11. [MemoryHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.MemoryHandler) instances send messages to a buffer in memory, which is flushed whenever specific criteria are met.
12. [HTTPHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.HTTPHandler) instances send messages to an HTTP server using either GET or POST semantics.
13. [WatchedFileHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.WatchedFileHandler) instances watch the file they are logging to. If the file changes, it is closed and reopened using the file name. This handler is only useful on Unix-like systems; Windows does not support the underlying mechanism used.
14. [QueueHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.QueueHandler) instances send messages to a queue, such as those implemented in the queue or multiprocessing modules.
15. [NullHandler](https://docs.python.org/3/library/logging.handlers.html#logging.NullHandler) instances do nothing with error messages. They are used by library developers who want to use logging, but want to avoid the ‘No handlers could be found for logger XXX’ message which can be displayed if the library user has not configured logging. See [Configuring Logging for a Library](https://docs.python.org/3/howto/logging.html#library-config) for more information.

## formatter

Once handler dealt with the python object it needs to convert that to
- log text
- custom json lines etc

its the formatter that determines what your log looks like
see log record attributes link

## custom logger

### logs

#### text

```
[DEBUG|main|L25] 2024-01-23T09:20:09-0600: debug message
[INFO|main|L26] 2024-01-23T09:20:09-0600: info message
[WARNING|main|L27] 2024-01-23T09:20:09-0600: warning message
[ERROR|main|L28] 2024-01-23T09:20:09-0600: error message
[CRITICAL|main|L29] 2024-01-23T09:20:09-0600: critical message
[ERROR|main|L33] 2024-01-23T09:20:09-0600: exception message
Traceback (most recent call last):
  File '/home/dukenukem/PycharmProjects/custom_logging/main.py', line 31, in main
    1 / 0
    ~~^~~
ZeroDivisionError: division by zero
```

#### json lines (jsonl)

```json
{"level": "DEBUG", "message": "debug message", "timestamp": "2024-01-23T15:20:53.151737+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 25, "thread_name": "MainThread", "x": "hello"}
{"level": "INFO", "message": "info message", "timestamp": "2024-01-23T15:20:53.151737+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 26, "thread_name": "MainThread"}
{"level": "WARNING", "message": "warning message", "timestamp": "2024-01-23T15:20:53.151737+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 27, "thread_name": "MainThread"}
{"level": "ERROR", "message": "error message", "timestamp": "2024-01-23T15:20:53.152735+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 28, "thread_name": "MainThread"}
{"level": "CRITICAL", "message": "critical message", "timestamp": "2024-01-23T15:20:53.152735+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 29, "thread_name": "MainThread"}
{"level": "ERROR", "message": "exception message", "timestamp": "2024-01-23T15:20:53.152735+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 33, "thread_name": "MainThread", "exc_info": "Traceback (most recent call last):\n  File '/home/dukenukem/PycharmProjects/custom_logging/main.py', line 31, in main\n    1 / 0\n    ~~^~~\nZeroDivisionError: division by zero"}
```

### main.py

- [atexit (exit handler)](https://docs.python.org/3/library/atexit.html)

```python
import atexit  
import json
import logging.config
import logging.handlers
import pathlib

logger = logging.getLogger("my_app")  # __name__ is a common choice


def setup_logging():
    config_file = pathlib.Path("configs/stderr.json")
    with open(config_file) as f_in:
        config = json.load(f_in)

    logging.config.dictConfig(config)
    queue_handler = logging.getHandlerByName("queue_handler")
    if queue_handler is not None:
        queue_handler.listener.start()
        atexit.register(queue_handler.listener.stop)


def main():
    setup_logging()
    logging.basicConfig(level="INFO")
    logger.debug("debug message", extra={"x": "hello"})
    logger.info("info message")
    logger.warning("warning message")
    logger.error("error message")
    logger.critical("critical message")
    try:
        1 / 0
    except ZeroDivisionError:
        logger.exception("exception message")


if __name__ == "__main__":
    main()
```
### mylogger.py

```python
import datetime as dt
import json
import logging
from typing import override

LOG_RECORD_BUILTIN_ATTRS = {
    "args",
    "asctime",
    "created",
    "exc_info",
    "exc_text",
    "filename",
    "funcName",
    "levelname",
    "levelno",
    "lineno",
    "module",
    "msecs",
    "message",
    "msg",
    "name",
    "pathname",
    "process",
    "processName",
    "relativeCreated",
    "stack_info",
    "thread",
    "threadName",
    "taskName",
}


class MyJSONFormatter(logging.Formatter):
    def __init__(self, *, fmt_keys: dict[str, str] | None = None):
        super().__init__()
        self.fmt_keys = fmt_keys if fmt_keys is not None else {}

    @override
    def format(self, record: logging.LogRecord) -> str:
        message = self._prepare_log_dict(record)
        return json.dumps(message, default=str)

    def _prepare_log_dict(self, record: logging.LogRecord):
        always_fields = {
            "message": record.getMessage(),
            "timestamp": dt.datetime.fromtimestamp(
                record.created, tz=dt.timezone.utc
            ).isoformat(),
        }
        if record.exc_info:
            always_fields["exc_info"] = self.formatException(record.exc_info)

        if record.stack_info:
            always_fields["stack_info"] = self.formatStack(record.stack_info)

        message = {
            key: msg_val
            if (msg_val := always_fields.pop(val, None)) is not None
            else getattr(record, val)
            for key, val in self.fmt_keys.items()
        }
        message.update(always_fields)

        for key, val in record.__dict__.items():
            if key not in LOG_RECORD_BUILTIN_ATTRS:
                message[key] = val

        return message

# keep only info and debug
# bool = process record or not
class NonErrorFilter(logging.Filter):
    @override
    def filter(self, record: logging.LogRecord) -> bool | logging.LogRecord:
        return record.levelno <= logging.INFO
```

### json configs

json is preferred over yaml as json can be imported natively in python and yaml is also prone to errors

#### stdout

```json
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "simple": {
      "format": "%(levelname)s: %(message)s"
    }
  },
  "handlers": {
    "stdout": {
      "class": "logging.StreamHandler",
      "formatter": "simple",
      "stream": "ext://sys.stdout"
    }
  },
  "loggers": {
    "root": {
      "level": "DEBUG",
      "handlers": [
        "stdout"
      ]
    }
  }
}
```

#### stderr

```json
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "simple": {
      "format": "%(levelname)s: %(message)s"
    },
    "detailed": {
      "format": "[%(levelname)s|%(module)s|L%(lineno)d] %(asctime)s: %(message)s",
      "datefmt": "%Y-%m-%dT%H:%M:%S%z"
    }
  },
  "handlers": {
    "stderr": {
      "class": "logging.StreamHandler",
      "level": "WARNING",
      "formatter": "simple",
      "stream": "ext://sys.stderr"
    },
    "file": {
      "class": "logging.handlers.RotatingFileHandler",
      "level": "DEBUG",
      "formatter": "detailed",
      "filename": "logs/my_app.log",
      "maxBytes": 10000,
      "backupCount": 3
    }
  },
  "loggers": {
    "root": {
      "level": "DEBUG",
      "handlers": [
        "stderr",
        "file"
      ]
    }
  }
}
```

#### stderror custom json

parsing json logs that consist of json lines:

```json
{"level": "DEBUG", "message": "debug message", "timestamp": "2024-01-23T15:20:53.151737+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 25, "thread_name": "MainThread", "x": "hello"}
{"level": "INFO", "message": "info message", "timestamp": "2024-01-23T15:20:53.151737+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 26, "thread_name": "MainThread"}
{"level": "WARNING", "message": "warning message", "timestamp": "2024-01-23T15:20:53.151737+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 27, "thread_name": "MainThread"}
{"level": "ERROR", "message": "error message", "timestamp": "2024-01-23T15:20:53.152735+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 28, "thread_name": "MainThread"}
{"level": "CRITICAL", "message": "critical message", "timestamp": "2024-01-23T15:20:53.152735+00:00", "logger": "my_app", "module": "main", "function": "main", "line": 29, "thread_name": "MainThread"}
```

```json
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "simple": {
      "format": "%(levelname)s: %(message)s",
      "datefmt": "%Y-%m-%dT%H:%M:%S%z"
    },
    "json": {
      "()": "mylogger.MyJSONFormatter",
      "fmt_keys": {
        "level": "levelname",
        "message": "message",
        "timestamp": "timestamp",
        "logger": "name",
        "module": "module",
        "function": "funcName",
        "line": "lineno",
        "thread_name": "threadName"
      }
    }
  },
  "handlers": {
    "stderr": {
      "class": "logging.StreamHandler",
      "level": "WARNING",
      "formatter": "simple",
      "stream": "ext://sys.stderr"
    },
    "file": {
      "class": "logging.handlers.RotatingFileHandler",
      "level": "DEBUG",
      "formatter": "json",
      "filename": "logs/my_app.log.jsonl",
      "maxBytes": 10000,
      "backupCount": 3
    }
  },
  "loggers": {
    "root": {
      "level": "DEBUG",
      "handlers": [
        "stderr",
        "file"
      ]
    }
  }
}
```
#### filtered stdout & stderr

```json
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "simple": {
      "format": "%(levelname)s: %(message)s",
      "datefmt": "%Y-%m-%dT%H:%M:%S%z"
    }
  },
  "filters": {
    "no_errors": {
      "()": "mylogger.NonErrorFilter"
    }
  },
  "handlers": {
    "stdout": {
      "class": "logging.StreamHandler",
      "formatter": "simple",
      "stream": "ext://sys.stdout",
      "filters": ["no_errors"]
    },
    "stderr": {
      "class": "logging.StreamHandler",
      "formatter": "simple",
      "stream": "ext://sys.stderr",
      "level": "WARNING"
    }
  },
  "loggers": {
    "root": {
      "level": "DEBUG",
      "handlers": [
        "stdout",
        "stderr"
      ]
    }
  }
}
```

#### queuehandler

non-blocking

##### stderr

```json
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "json": {
      "()": "mylogger.MyJSONFormatter",
      "fmt_keys": {
        "level": "levelname",
        "message": "message",
        "timestamp": "timestamp",
        "logger": "name",
        "module": "module",
        "function": "funcName",
        "line": "lineno",
        "thread_name": "threadName"
      }
    }
  },
  "handlers": {
    "stderr": {
      "class": "logging.StreamHandler",
      "level": "WARNING",
      "formatter": "json",
      "stream": "ext://sys.stderr"
    },
    "queue_handler": {
      "class": "logging.handlers.QueueHandler",
      "handlers": [
        "stderr"
      ],
      "respect_handler_level": true
    }
  },
  "loggers": {
    "root": {
      "level": "DEBUG",
      "handlers": [
        "queue_handler"
      ]
    }
  }
}
```

##### stderr json lines

```json
{
  "version": 1,
  "disable_existing_loggers": false,
  "formatters": {
    "simple": {
      "format": "[%(levelname)s|%(module)s|L%(lineno)d] %(asctime)s: %(message)s",
      "datefmt": "%Y-%m-%dT%H:%M:%S%z"
    },
    "json": {
      "()": "mylogger.MyJSONFormatter",
      "fmt_keys": {
        "level": "levelname",
        "message": "message",
        "timestamp": "timestamp",
        "logger": "name",
        "module": "module",
        "function": "funcName",
        "line": "lineno",
        "thread_name": "threadName"
      }
    }
  },
  "handlers": {
    "stderr": {
      "class": "logging.StreamHandler",
      "level": "WARNING",
      "formatter": "simple",
      "stream": "ext://sys.stderr"
    },
    "file_json": {
      "class": "logging.handlers.RotatingFileHandler",
      "level": "DEBUG",
      "formatter": "json",
      "filename": "logs/my_app.log.jsonl",
      "maxBytes": 10000,
      "backupCount": 3
    },
    "queue_handler": {
      "class": "logging.handlers.QueueHandler",
      "handlers": [
        "stderr",
        "file_json"
      ],
      "respect_handler_level": true
    }
  },
  "loggers": {
    "root": {
      "level": "DEBUG",
      "handlers": [
        "queue_handler"
      ]
    }
  }
}
```
