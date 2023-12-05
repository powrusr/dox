# sphinx

## installation

### create requirements file

```bash
touch requirements.txt;echo "sphinx\nmyst-parser\nsphinx-autobuild\npython-docs-theme" >> requirements.txt
```

\nrstcheck

### create environment

mkvirtualenv dox -r requirements.txt

verify package installation

```bash
which sphinx-quickstart 
/home/user/.virtualenvs/dox/bin/sphinx-quickstart
```

### quickstart sphinx

check [here](https://www.sphinx-doc.org/en/master/usage/quickstart.html) for all options

--ext-githubpages creates .nojekyll file on generated HTML directory to publish the document on GitHub Pages
--extensions EXTENSIONS enable arbitrary extensions
--project PROJECT project name
--author AUTHOR  author names
-v VERSION version of project
-release VERSION release of project
--sep if specified, separate source and build dirs


```bash
sphinx-quickstart --project "PowrUsr Docs" --author "PowrUsr" --release "alpha" -v "2023-12-05" \
--ext-githubpages --extensions "myst_parser" --sep --language en
```

### more on myst parser:

You can further configure MyST-Parser to allow custom syntax that standard CommonMark doesn’t support.
Read more in the MyST-Parser [documentation](https://myst-parser.readthedocs.io/en/latest/using/syntax-optional.html)

## conf.py configuration


### change default html theme

```bash
# test
sed -n "s/alabaster/python_docs_theme/p" source/conf.py

# apply
sed -i "s/alabaster/python_docs_theme/" source/conf.py
```

remove redundant text if you like in index file
```bash
# test
sed -E -n "N;s/You can adapt/lol/p" source/index.rst
   lol this file completely to your liking, but it should at least
   contain the root `toctree` directive.

# apply
sed -E -i "N;s/You can adapt.*//" source/index.rst
```

### define file extensions

```bash
echo -e "source_suffix = {\n\
    '.rst': 'restructuredtext',\n\
    '.txt': 'markdown',\n\
    '.md': 'markdown',\n\
> }">> source/conf.py
```

### gitignore update

ensure correct docs/build exclusion instead of docs/_build in gitignore file

```bash
# test
sed -n "s#docs/_build/#docs/build/#p" .gitignore 
docs/build/

# apply
sed -i "s#docs/_build/#docs/build/#" .gitignore
```

Also ignore the Makefile and make.bat files as they are
created upon running `sphinx-quickstart`

```bash
echo -e "Makefile\nmake.bat\n" >> .gitignore 
```

## gh-pages prep

create a gh-pages branch

`git branch gh-pages`

### convert build to docs

change make.bat and Makefile builddir variable to docs

**in github you can't select the build folder, only / or docs**
so it is best to name build directory as docs to automate things


The following sed command will detect the two lines that need to change

```bash
$ sed -E -n "/=\s?build/p" Makefile make.bat 
BUILDDIR      = build
set BUILDDIR=build
```

### change Makefile:

```bash
# test
sed -E -n "s/^(BUILDDIR\s*=\s*)(build)/\1docs/p" Makefile
BUILDDIR      = docs

# apply
sed -E -i "s/^(BUILDDIR\s*=\s*)(build)/\1docs/" Makefile
```

### change make.bat:

```bash
# test
sed -E -n "s/(set BUILDDIR=)(build)/\1docs/p" make.bat

# apply
sed -E -i "s/(set BUILDDIR=)(build)/\1docs/" make.bat
```

### rename build to docs

`mv build docs`

### glob in source/index.rst

```rst

.. toctree::
   :maxdepth: 4
   :glob:
   :caption: Contents:

   sphinx
   bash/*

```

