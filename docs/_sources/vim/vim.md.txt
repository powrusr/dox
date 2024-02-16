# vim

## links

[vim docs](https://vimdoc.sourceforge.net/)

## repeatable actions

| Intent                           | Act                   | Repeat | Reverse |
|----------------------------------|-----------------------|--------|---------|
| Make a change                    | {edit}                | .      | u       |
| Scan line for next character     | f{char} / t{char}     | ;      | ,       |
| Scan line for previous character | F{char} / T{char}     | ;      | ,       |
| Scan document for next match     | /pattern <CR>         | n      | N       |
| Scan document for previous match | ?pattern <CR>         | n      | N       |
| Substitute                       | :s/target/replacement | &      | u       |
| Execute sequence of changes      | qx{changes}q          | @x     | u       |



## plugins

since version 8 you can install packages inside `~/.vim/pack`

A package can hold plugins in two different directories, **start** and **opt**. The plugins under start/ folder are loaded on startup, while the plugins under opt/ folder are loaded manually by the user with the command `:packadd`

```bash
# Plugin auto loaded on startup
~/.vim/pack/*/start/{name}

# Plugin loaded manually, with :packadd {name}
~/.vim/pack/*/opt/{name}
```

git clone into these directories directly

```bash
mkdir -p ~/.vim/pack/tpope/start
cd ~/.vim/pack/tpope/start
git clone https://tpope.io/vim/surround.git
vim -u NONE -c "helptags surround/doc" -c q
```

### surround

- change surrounding `'word'` to `"word"`:  `cs'"`  
  > mnemonic: change surrounding to '
- change surrounding `"word"` to `<p>word</p>`:  `cs"<p>`  
  > mnemonic: change surrounding " to typed tag   
- change surrounding `<p>word</p>` to `"word":` `cst"`  
  > mnemonic: change surrounding tag to "   
- remove surrounding `'word'` to `word:` `ds'`  
  > mnemonic: delete surrounding '   
- change surrounding `word` to `[word]:`  `ysiw]` iw = text object  
  > mnemonic: yield surounding ] to text object   
- surround entire line with parentheses:  `yss)`  
  > mnemonic: yield supersurround )   
- remove surroundings on `({ Hello } world!)`:  `ds)ds}`  
  > mnemonic: delete surrounding ) delete surrounding }   
- change `word` to `<em>word</em>`:  `ysiw<em>`  
  > mnemonic: yields surrounding iw (text object) <em>   
- change `<p>word</p>` to `` `<p>word</p>` ``:  `` ysat` `` at=a tag block   
  > mnemonic: yield surround a tag with `  see `:h at`   
- change `some any|selectionO` to `some "any|selectionO"`: `vt c""<ESC><P>`  
  > mnemonic: visually select till space change (selection is copied to buffer) "" escape paste selection before character   

  
How do I surround without adding a space?

Only the opening brackets `[ { (` add a space. Use a closing bracket, or the `b (()` and `B ({)` aliases


## magical

from `:help pattern`

- Use of `\m` makes the pattern after it be interpreted as if 'magic' is set,  ignoring the actual value of the 'magic' option  
- Use of `\M` makes the pattern after it be interpreted as if 'nomagic' is used.
- Use of `\v` means that in the pattern after it all ASCII characters except `0-9`, `a-z`, `A-Z` and `_` have a special meaning. "very magic"
- Use of `\V` means that in the pattern after it only the backslash and the terminating character (/ or ?) has a special meaning. "very nomagic"
 

eg match word followed by a comma without having to escape special characters with `\v` at beginning of pattern:

```vim
:%s#\v\w+,##gc
```

## lookarounds

### preceeded by
- don't match if preceeded by with `@<!`
- match if preceeded by with `@<=`

### followed by
- don't match if followed by with `@!`
- match if followed by with `@=` or `&`

eg don't match if a word is preceeded by a pipe symbol

```vim
:%s#\v(|)@<!(__\w+__)#`\2`#gc
```

eg match if followed by a pipe symbol

```vim
:%s#\v(__\w+__)(\|)@=#`\2`#gc
```

see help in vim `:h perl-patterns`
```vim
Capability                      in Vimspeak     in Perlspeak ~
----------------------------------------------------------------
force case insensitivity        \c              (?i)
force case sensitivity          \C              (?-i)
backref-less grouping           \%(atom\)       (?:atom)
conservative quantifiers        \{-n,m}         *?, +?, ??, {}?
0-width match                   atom\@=         (?=atom)
0-width non-match               atom\@!         (?!atom)
0-width preceding match         atom\@<=        (?<=atom)
0-width preceding non-match     atom\@<!        (?<!atom)
match without retry             atom\@>         (?>atom)
```

## patterns

[pattern docs](https://vimdoc.sourceforge.net/htmldoc/pattern.html#pattern)

- word begin: \<
- word end: \>
- use second match as replacement: \2

```|`Mapping`  |`Collection`   __getitem__, _```

```bash
:%s#\(`\)\(\<\w\+\>\)\(`\)#\2#gc
```
use
```vim
:%s#\v(__\w+__)(`)@!#`\1`#g
```
to turn 
```
|Iterator |Iterable|__next__|__iter__|
|Generator  |Iterator|send, throw|close, __iter__,__iter__, __next__|
```
into
```
|Iterator |Iterable|__next__|__iter__|
|Generator  |Iterator|send, throw|close, `__iter__`,`__iter__`, `__next__`|
```
