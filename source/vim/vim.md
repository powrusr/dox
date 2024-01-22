# vim
[vim docs](https://vimdoc.sourceforge.net/)

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
