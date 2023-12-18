# code library

## card game

utils/card.py
```python
class Card:

    def __init__(self, color, composition, name, points):
        self._color = color
        self._composition = composition
        self._name = name
        self._points = points

    def __call__(self, *args, **kwargs):
        return f"new card: {self._color} {self._composition} {self._name} {self._points} was created"

    def __str__(self):
        return f'[{self._color} {self._composition} {self._name}: {self._points}]'

    def __repr__(self):
        return f'[{self._color} {self._composition} {self._name}: {self._points}]'

    @property
    def get_card_properties(self):
        return self._color, self._composition, self._name, self._points
```

game.py
```python
import pprint as pp
import random
import typing
from utils.card import Card


def shuffle_cards(deck_of_cards: list):
    random.shuffle(deck_of_cards)


def generate_card_instances():
    card_properties = {"club": "black",
                       "diamond": "red",
                       "heart": "red",
                       "spade": "black"}
    cards = []  # how to initialize this as a list of Card without creating an instance?
    cards_in_game = {
        "ace": 1, "two": 2, "three": 3, "four": 4,
        "five": 5, "six": 6, "seven": 7, "eight": 8,
        "nine": 9, "ten": 10, "jack": 10, "queen": 10,
        "king": 10
    }
    card_number = 0
    card_deck = {
        card_number: {"color": "",          # "black" "spade" "jack" 10
                      "composition": "",
                      "name": "",
                      "points": 0
                      }
    }
    for card, points in cards_in_game.items():    # for every card: "ace" with points: 1 you have ----
        for composition, color in card_properties.items():  # ----> 4 compositions of which 2 are black & 2 are red
            card_deck[card_number] = {"color": color,
                                      "composition": composition,  # filling this card_deck list with dictionaries
                                      "name": card,
                                      "points": points}
            cards.append(Card(color, composition, card, points))
            card_number += 1
    return cards


if __name__ == '__main__':
    all_cards = generate_card_instances()
    print(all_cards)
    """ [[black club ace: 1], [red diamond ace: 1], [red heart ace: 1], [black spade ace: 1], ... """
    shuffle_cards(all_cards)
    print(all_cards)
    """ [[red heart three: 3], [black club ten: 10], [black spade queen: 10], [red heart ten: 10], """
    pp.pprint([card.get_card_properties for card in all_cards])
```

## adding files to a list & compression

```python
from pathlib import Path
import contextlib
import os


def file_exists(file_path):
    return Path(file_path).exists()


def compress_to_bz2_file(source_file, output_file, compression_lvl=9):
    import bz2
    if 1 <= compression_lvl <= 9:
        with open(source_file, 'rb') as data:
            tar_bz2_contents = bz2.compress(data.read(), compresslevel=compression_lvl)
    else:
        raise ValueError("compression level has to be between 1 and 9")


# concatenate_files_in_list(text_files_list, final_file, 'latin-1', True)
def concatenate_files_in_list(file_list, output_file, encode_in='utf-8', is_small=False, use_shutil=False):
    # don't forget to specify target folder when opening files!
    # todo: check if file was created already
    encoding_format = encode_in
    print(f"target file -> {output_file}")
    readmode = ('rb' if use_shutil else 'r')
    writemode = ('wb' if use_shutil else 'w+')
    with open(output_file, writemode) as finaltext:
        for file_name in file_list:
            print(f"currently appending file: {file_name}")
            with open(file_name, readmode, encoding=encoding_format) as file_currently_opened:
                if use_shutil:
                    import shutil
                    shutil.copyfileobj(file_name, finaltext)
                elif is_small:
                    finaltext.write(file_currently_opened.read())
                elif not is_small:
                    for line in file_currently_opened:
                        finaltext.write(line)


def add_files_from_folder_to_list(filetype, folder_target='./'):
    folder = set_folder(folder_target)
    print(folder)
    print(f"getting files from {folder} of type {filetype} ...")
    all_files = []
    for path, dirs, files in os.walk(folder):
        for filename in files:
            if filename.endswith('.txt'):
                full_path = os.path.join(path,filename)
                all_files.append(full_path)
    with contextlib.suppress(Exception):
        raise RuntimeError('something went wrong')
    return all_files


def set_folder(folder_dir):
    isdir = Path(folder_dir)
    if not isdir:
        raise Exception(f"folder you're trying to set is invalid -> {folder_dir}")
    absolute_path = isdir.resolve()
    return absolute_path


def test_files_in_list(file_list: []):
    print("testing files in list... ")
    for file in file_list:
        if not Path(file).exists():
            print(f"oops: {file} doesn't seem to exist")
```

## logging

### fn logger

```python
class FunctionLogger(object):

    def __init__(self, function):
        self.filename = function.__name__ + ".log"
        self.logger = logging.getLogger(__name__)
        self.logger.setLevel(logging.INFO)
        self.fh = logging.FileHandler(self.filename)
        self.fh.setLevel(logging.INFO)
        self.formatter = logging.Formatter('%(asctime)s :: %(levelname)s :: %(message)s')


        self.fh.setFormatter(self.formatter)
        self.logger.addHandler(self.fh)
        self.original_function = function

    def __call__(self, *args, **kwargs):
        print(f"this code adds functionality before {self.original_function.__name__}")
        self.logger.info(f"running {self.original_function.__name__} with arguments {args} and {kwargs}")
        return self.original_function(*args, **kwargs)  # execute
```
