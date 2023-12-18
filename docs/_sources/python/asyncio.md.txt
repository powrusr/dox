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

## websockets


```python

```
