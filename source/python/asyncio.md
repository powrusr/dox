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
## other examples

```python
import aiohttp
import asyncio
import urllib.parse
import datetime

async def get(session, url):
    print("[{:%M:%S.%f}] getting {} ...".format(datetime.datetime.now(), urllib.parse.urlsplit(url).hostname))
    async with session.get(url) as resp:
        print("[{:%M:%S.%f}] {}, status: {}".format(datetime.datetime.now(), urllib.parse.urlsplit(url).hostname, resp.status))
        doc = await resp.text()
        print("[{:%M:%S.%f}] {}, len: {}".format(datetime.datetime.now(), urllib.parse.urlsplit(url).hostname, len(doc)))

async def main():
    session = aiohttp.ClientSession()

    url = "http://demo.borland.com/Testsite/stadyn_largepagewithimages.html"
    f1 = asyncio.ensure_future(get(session, url))
    print("[{:%M:%S.%f}] added {} to event loop".format(datetime.datetime.now(), urllib.parse.urlsplit(url).hostname))

    url = "https://stackoverflow.com/questions/46445019/aiohttp-when-is-the-response-status-available"
    f2 = asyncio.ensure_future(get(session, url))
    print("[{:%M:%S.%f}] added {} to event loop".format(datetime.datetime.now(), urllib.parse.urlsplit(url).hostname))

    url = "https://api.github.com/events"
    f3 = asyncio.ensure_future(get(session, url))
    print("[{:%M:%S.%f}] added {} to event loop".format(datetime.datetime.now(), urllib.parse.urlsplit(url).hostname))

    await f1
    await f2
    await f3

    session.close()

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())
```

```console
[16:42.415481] added demo.borland.com to event loop
[16:42.415481] added stackoverflow.com to event loop
[16:42.415481] added api.github.com to event loop
[16:42.415481] getting demo.borland.com ...
[16:42.422481] getting stackoverflow.com ...
[16:42.682496] getting api.github.com ...
[16:43.002515] demo.borland.com, status: 200
[16:43.510544] stackoverflow.com, status: 200
[16:43.759558] stackoverflow.com, len: 110650
[16:43.883565] demo.borland.com, len: 239012
[16:44.089577] api.github.com, status: 200
[16:44.318590] api.github.com, len: 43055
```


## websockets


```python

```
