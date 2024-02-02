# asyncio

## async but not concurrent

```python
import aiohttp
import asyncio
import time

start_time = time.time()

async def main():

    async with aiohttp.ClientSession() as session:

        for number in range(1, 151):
            pokemon_url = f'https://pokeapi.co/api/v2/pokemon/{number}'
            async with session.get(pokemon_url) as resp:
                pokemon = await resp.json()
                print(pokemon['name'])

asyncio.run(main())
print("--- %s seconds ---" % (time.time() - start_time))
```
```output
bulbasaur
ivysaur
venusaur
...
dratini
dragonair
dragonite
mewtwo
--- 6.3120012283325195 seconds ---
```

## async & concurrent

When we await this call to asyncio.gather, we get back an iterable for all of the futures that were passed in, maintaining their order in the list. This way we're only awaiting one time

```python
import aiohttp
import asyncio
import time

start_time = time.time()


async def get_pokemon(session, url):
    async with session.get(url) as resp:
        pokemon = await resp.json()
        return pokemon['name']


async def main():

    async with aiohttp.ClientSession() as session:

        tasks = []
        for number in range(1, 151):
            url = f'https://pokeapi.co/api/v2/pokemon/{number}'
            tasks.append(asyncio.ensure_future(get_pokemon(session, url)))

        original_pokemon = await asyncio.gather(*tasks)
        for pokemon in original_pokemon:
            print(pokemon)

asyncio.run(main())
print("--- %s seconds ---" % (time.time() - start_time))
```
```output
bulbasaur
ivysaur
venusaur
charmander
charmeleon
...
dratini
dragonair
dragonite
mewtwo
--- 3.4634764194488525 seconds ---
```
## aiohttp

[aiohttp docs](https://docs.aiohttp.org/en/stable/index.html)

```python
```


## websockets


```python

```

## producer-consumer

### consumer.py

```python
"""The actual task 'worker' or 'processor' - the work queue consumer"""
import asyncio
from random import random
from time import perf_counter


async def do_work(work_queue: asyncio.Queue, result_queue: asyncio.Queue) -> None:
    """
    This function (coroutine) will perform the actual work, by pulling an item
    from the work queue, doing some work, and pushing the result
    to the result queue, and repeating indefinitely.

    This coroutine never terminates itself - it just keeps looking for work to
    do in the work_queue. It will instead be terminated by the controller when
    the controller decides all work has been done.

    :param work_queue: the work queue the task consumes (pulls from)
    :param result_queue: the result queue the task pushes a result to
        once the work is complete
    :return:
    """

    while True:
        # grab an item from the queue (if there is one)
        task_data = await work_queue.get()

        # read the data we need to perform the work
        task_id = task_data["task_id"]
        number = task_data["number"]

        # do some work that takes some time - simulated here with an async sleep
        start = perf_counter()
        await asyncio.sleep(random() * 2)  # random wait time up to 2 seconds
        result = number * number
        end = perf_counter()

        # push result to result queue
        await result_queue.put(
            {
                "task_id": task_id,
                "result": result,
                "time_secs": end - start
            }
        )

        # inform work queue the task is complete
        work_queue.task_done()
```

### producer.py

```python
"""This code defines the producer, which populates the work queue"""
import asyncio
from typing import List


async def produce_work(
    batch: List[dict], work_queue: asyncio.Queue, producer_completed: asyncio.Event
):
    """
    Puts all the requested work into the work queue.

    :param batch: list of all the params that were submitted to be processed (this is the list of
        dicts the main application sent over for processing)
    :param work_queue: main work queue that contains each individual task params
    :param producer_completed: event to indicate the producer has finished producing
        all requested work
    :return:
    """
    for data in batch:
        await work_queue.put(data)

    # finished putting all the data into the work queue - indicate we are done using
    # the producer_completed event
    producer_completed.set()
```

### resulthandler.py

```python
"""Contains the code to handle results waiting in the result queue"""
import asyncio
from typing import Callable


async def handle_task_result(result_queue: asyncio.Queue, callback: Callable[[dict], None]):
    """Result item handler
    This function (coroutine) will handle any results sitting in the result queue by
    issuing the callback.

    Just like the task worker, this coroutine never terminates itself. It will instead
    be terminated by the controller when the controller decides all work has been done.

    :param result_queue: the result queue this task consumes (pulls from)
    :param callback: the callback function to call with the results pulled from the queue
    :return:
    """
    while True:
        result = await result_queue.get()
        callback(result)
        result_queue.task_done()  # tell the queue we are done with the item
```

### controller.py

```python
"""Main Controller

This is the main controller that orchestrates the producer, the 'workers' (aka tasks),
sets up the various queues, and tracks when all work is completed to shut down everything
and issue the final job completed callback.
"""
import asyncio
from time import perf_counter
from typing import Callable, List

import consumer
import producer
import resulthandler


# some constants, but could be defined in a config file, or simply passed to run_job function when called
NUM_WORKERS = 25
WORK_QUEUE_MAX_SIZE = 100

NUM_RESULT_HANDLERS = 10
RESULT_QUEUE_MAX_SIZE = 100


async def _controller(
        batch: List[dict], task_completed_callback: Callable, job_completed_callback: Callable
) -> None:
    """
    This is the async controller.

    :param batch: a list of dictionaries received that defines the parameters for each task that has to run
    :param task_completed_callback: the callback to use when each task result becomes available
    :param job_completed_callback: the callback to use when the overall job is completed
    :return:
    """
    start = perf_counter()

    # create the work and result queues
    work_queue = asyncio.Queue(maxsize=WORK_QUEUE_MAX_SIZE)
    result_queue = asyncio.Queue(maxsize=RESULT_QUEUE_MAX_SIZE)

    # create a list of all the tasks that will need to run async
    tasks = []

    # Define the producer task, defining the event we'll look for when the producer is done
    producer_completed = asyncio.Event()
    producer_completed.clear()  # set the event status to False for starters
    tasks.append(
        asyncio.create_task(producer.produce_work(batch, work_queue, producer_completed))
    )

    # Create the worker (consumer) tasks
    for _ in range(NUM_WORKERS):
        tasks.append(
            asyncio.create_task(consumer.do_work(work_queue, result_queue))
        )

    # Create the result handler tasks
    for _ in range(NUM_RESULT_HANDLERS):
        tasks.append(
            asyncio.create_task(resulthandler.handle_task_result(result_queue, task_completed_callback))
        )

    # Now wait completion of producer, and kick off the consumers and result handlers
    await producer_completed.wait()
    await work_queue.join()
    await result_queue.join()

    # once we reach here, we're all done, so cancel all tasks
    for task in tasks:
        task.cancel()

    end = perf_counter()

    # all done, callback using the provided callback function
    job_completed_callback({"elapsed_secs": end - start})


def run_job(
    batch: List[dict], task_completed_callback: Callable, job_completed_callback: Callable
) -> None:
    """
    This is the function caller calls to kick off the job.
    Note that this function is not a coroutine - it's a standard function that will run the top-level
    entry point for our async processing.

    :param batch: a list of dictionaries received that defines the parameters for each task that has to run
    :param task_completed_callback: the callback to use when each task result becomes available
    :param job_completed_callback: the callback to use when the overall job is completed
    :return:
    """
    asyncio.run(_controller(batch, task_completed_callback, job_completed_callback))
```

### main.py

```python
"""This is the main controller for the producer/consumer

Here we'll define
        - the parameters for each task that needs to run,
        - the callback handler that will handle when each task runs
            to completion
        - the callback handler that will handle the message that all tasks
            have completed

And finally, we'll kick off the process using the main() function.
"""
from functools import partial
from random import seed
from uuid import uuid4

from controller import run_job


def main(job_id: str) -> None:
    """
    Main app that kicks a "job" that consists of running multiple tasks

    :param job_id: some job identifier
    :return:
    """

    print(f"Starting Job {job_id}")

    # define callbacks, "injecting" the job_id
    task_callback = partial(task_completed_callback_handler, job_id)
    job_callback = partial(job_completed_callback_handler, job_id)

    # define the parameters for the tasks that will need to run
    task_data = [
        {"task_id": i, "number": i}
        for i in range(10)
    ]

    # start job
    run_job(task_data, task_callback, job_callback)


def task_completed_callback_handler(job_id: str, callback_message: dict) -> None:
    print(f"Task completed in {job_id=}: {callback_message=}")


def job_completed_callback_handler(job_id: str, callback_message: dict) -> None:
    print(f"Job {job_id} completed: {callback_message=}")


if __name__ == '__main__':
    seed(0)  # just to get repeatability in various sleeps we use to simulate long-running processes
    main(str(uuid4()))
```
