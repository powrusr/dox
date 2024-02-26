# pyenv

- [pyenv docs](https://github.com/pyenv/pyenv)
- [pyenv virtual-env docs](https://github.com/pyenv/pyenv-virtualenv)

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

- pyenv commands: This command renders a list of all the commands and subcommands you can use with pyenv. 
- pyenv global: You use this command to set the global Python version that it’ll use in all shells. Using the command will compose the version name into the ~/.pyenv/version file. It is possible to override the global Python versions with an application-specific .python version file. Alternatively, you can also specify the environment variable PYENV_VERSION to do this.
- pyenv help: It outputs a list of all the commands you can use with pyenv, coupled with short explanations of what the commands are meant to accomplish. If you ever need help with the details of what a command does, this is the command you should run. 
- pyenv install: We’ve covered how to use the install command earlier in this post when listing Python versions with pyenv. The command allows you to install a specific Python version, and you can use the following flag attributes with it:
  -l: It shows you a list of all the installable Python versions.
  -g: You can build a debug Python version with this flag.
  -v: It turns on verbose mode, allowing you to print the compilation status to stdout.
- pyenv local: The local command allows you to set a local Python version specific to applications. It does this by supplying the appropriate version name to the .python version file. The command allows you to override the global Python version by setting the PYENV_VERSION or using the pyenv shell command. 
- pyenv shell: It enables you to set a Python version only for the shell using the PYENV_VERSION environment variable within the shell. The command will override not only the global version, but also the application-specific Python version.  
- pyenv versions: You can use this command to display all the Python versions currently installed on the machine. 
- pyenv which: This is the command that allows you to find the complete path to your machine’s executable. As mentioned earlier, pyenv leverages the shims, and the command allows you to see the path where the executable pyenv is running.


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

### test python version

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
3.12.1/envs/demo    3.12.1/envs/projectname  --help                   demo                projectname              --unset
duke@nukem:/home$ pyenv activate projectname 
(projectname) duke@nukem:/home$ pyenv deactivate
duke@nukem:/home$ pyenv virtualenv
virtualenv         virtualenv-delete  virtualenv-init    virtualenv-prefix  virtualenvs        
duke@nukem:/home$ pyenv virtualenv-delete projectname
pyenv-virtualenv: remove /home/duke/.pyenv/versions/3.12.1/envs/projectname? (y/N) y

# set back to system in local
(demo) duke@nukem:~/.pyenv/versions (master)$ pyenv local system
duke@nukem:~/.pyenv/versions (master)$ which python3
/home/duke/.pyenv/shims/python3
duke@nukem:~/.pyenv/versions (master)$ python3 -V
Python 3.10.12
duke@nukem:~/.pyenv/versions (master)$ pyenv versions
* system (set by /home/duke/.pyenv/versions/.python-version)
  3.12.1
  3.12.1/envs/demo
  demo --> /home/duke/.pyenv/versions/3.12.1/envs/demo
```

```bash
(demo) duke@nukem:~/gh/vega-protocol-market-maker-python (main)$ pyenv version
demo (set by /home/duke/.python-version)
(demo) duke@nukem:~/gh/vega-protocol-market-maker-python (main)$ pyenv versions
  system
  3.12.1
  3.12.1/envs/demo
* demo --> /home/duke/.pyenv/versions/3.12.1/envs/demo (set by /home/duke/.python-version)
(demo) duke@nukem:~/gh/vega-protocol-market-maker-python (main)$ pyenv virtualenv 3.12.1 vega-3.12
(demo) duke@nukem:~/gh/vega-protocol-market-maker-python (main)$ pyenv virtualenvs
  3.12.1/envs/demo (created from /home/duke/.pyenv/versions/3.12.1)
  3.12.1/envs/vega-3.12 (created from /home/duke/.pyenv/versions/3.12.1)
* demo (created from /home/duke/.pyenv/versions/3.12.1)
  vega-3.12 (created from /home/duke/.pyenv/versions/3.12.1)
(demo) duke@nukem:~/gh/vega-protocol-market-maker-python (main)$ pyenv activate vega-3.12 
(vega-3.12) duke@nukem:~/gh/vega-protocol-market-maker-python (main)$ 
```

### use latest python

```bash
duke@nukem:~$ pyenv install --list | grep -E '^\s+3.1[0-9]+'
  3.10.0
  3.10-dev
  3.10.1
  3.10.2
  3.10.3
  3.10.4
  3.10.5
  3.10.6
  3.10.7
  3.10.8
  3.10.9
  3.10.10
  3.10.11
  3.10.12
  3.10.13
  3.11.0
  3.11-dev
  3.11.1
  3.11.2
  3.11.3
  3.11.4
  3.11.5
  3.11.6
  3.11.7
  3.11.8
  3.12.0
  3.12-dev
  3.12.1
  3.12.2
  3.13.0a3
  3.13-dev
duke@nukem:~$ pyenv install -v 3.13-dev
```
create new env & setting it as default in python-version file
```bash
duke@nukem:~$ pyenv virtualenv 3.13-dev vega-voting
duke@nukem:~$ vim ~/.python-version 
3.13-dev
duke@nukem:~$ pyenv versions
  system
  3.12.1
  3.12.1/envs/liquidity
  3.12.1/envs/vega-3.12
* 3.13-dev (set by /home/duke/.python-version)
  3.13-dev/envs/vega-voting
  liquidity --> /home/duke/.pyenv/versions/3.12.1/envs/liquidity
  vega-3.12 --> /home/duke/.pyenv/versions/3.12.1/envs/vega-3.12
  vega-voting --> /home/duke/.pyenv/versions/3.13-dev/envs/vega-voting
duke@nukem:~$ pyenv which
duke@nukem:~$ pyenv which python
/home/duke/.pyenv/versions/3.13-dev/bin/python
```


