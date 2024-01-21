# pyenv

[pyenv docs](https://github.com/pyenv/pyenv)

## install

install dependencies

for more info:
https://devguide.python.org/getting-started/setup-building/index.html#build-dependencies

```bash
sudo apt-get install build-essential gdb lcov pkg-config \
      libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev \
      libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev \
      lzma lzma-dev tk-dev uuid-dev zlib1g-dev
```
```bash
curl https://pyenv.run | bash
```
add, as printed out after finishing installation, to .bashrc


```bash
# Load pyenv automatically
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

# Load pyenv-virtualenv automatically
eval "$(pyenv virtualenv-init -)"
```

check if desired python version is available

```bash
pyenv install --list | grep 3.12
  3.12.0
  3.12-dev
  3.12.1
  pypy2.7-7.3.12-src
  pypy2.7-7.3.12
  pypy3.9-7.3.12-src
  pypy3.9-7.3.12
  pypy3.10-7.3.12-src
  pypy3.10-7.3.12
```
install it

```bash
pyenv install -v 3.12.1
```
list your installed versions

```bash
ls ~/.pyenv/versions/
3.12.1
```
remove a python version

```bash
rm -rf ~/.pyenv/versions/3.12.1/
# or
pyenv uninstall 3.12.1
```
## use

```bash
which python
/home/duke/.pyenv/shims/python

pyenv versions
* system (set by /home/duke/.pyenv/version)
  3.12.1

duke@nukem:~$ python3 -V
Python 3.10.12
duke@nukem:~$ pyenv global 3.12.1
duke@nukem:~$ python3 -V
Python 3.12.1
```

if you like you can test your python version
```bash
python3 -m test
== CPython 3.12.1 (main, Jan 20 2024, 17:35:07) [GCC 11.4.0]
== Linux-6.5.0-14-generic-x86_64-with-glibc2.35 little-endian
== Python build: release shared
== cwd: /tmp/test_python_worker_47203æ
== CPU count: 4
== encodings: locale=UTF-8 FS=utf-8
== resources: all test resources are disabled, use -u option to unskip tests
..
0:13:16 load avg: 1.25 [467/469] test_zipimport_support
0:13:17 load avg: 1.23 [468/469] test_zlib
0:13:18 load avg: 1.23 [469/469] test_zoneinfo

== Tests result: SUCCESS ==
..

12 tests skipped (resource denied):
    test_curses test_ossaudiodev test_smtpnet test_socketserver
    test_tix test_tkinter test_ttk test_urllib2net test_urllibnet
    test_winsound test_xmlrpc_net test_zipfile64

440 tests OK.

Total duration: 13 min 19 sec
Total tests: run=39,148 skipped=1,259
Total test files: run=457/469 skipped=17 resource_denied=12
Result: SUCCESS
```

### manage envs

```bash
duke@nukem:/home$ pyenv virtualenv --help
Usage: pyenv virtualenv [-f|--force] [VIRTUALENV_OPTIONS] [version] <virtualenv-name>
       pyenv virtualenv --version
       pyenv virtualenv --help

  -f/--force       Install even if the version appears to be installed already. Skip
                   prompting for confirmation

Notable VIRTUALENV_OPTIONS passed to venv-creating executable, if applicable:
  -u/--upgrade     Imply --force

duke@nukem:/home$ pyenv virtualenv 3.12.1 projectname
duke@nukem:/home$ pyenv activate 
3.12.1/envs/liquidity    3.12.1/envs/projectname  --help                   liquidity                projectname              --unset
duke@nukem:/home$ pyenv activate projectname 
(projectname) duke@nukem:/home$ pyenv deactivate
duke@nukem:/home$ pyenv virtualenv
virtualenv         virtualenv-delete  virtualenv-init    virtualenv-prefix  virtualenvs        
duke@nukem:/home$ pyenv virtualenv-delete projectname
pyenv-virtualenv: remove /home/duke/.pyenv/versions/3.12.1/envs/projectname? (y/N) y

# set back to system in local
(liquidity) duke@nukem:~/.pyenv/versions (master)$ pyenv local system
duke@nukem:~/.pyenv/versions (master)$ which python3
/home/duke/.pyenv/shims/python3
duke@nukem:~/.pyenv/versions (master)$ python3 -V
Python 3.10.12
duke@nukem:~/.pyenv/versions (master)$ pyenv versions
* system (set by /home/duke/.pyenv/versions/.python-version)
  3.12.1
  3.12.1/envs/liquidity
  liquidity --> /home/duke/.pyenv/versions/3.12.1/envs/liquidity

```

