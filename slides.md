# Exploring <br> Virtual Environments

## with Brian Rutledge

- [What is a virtual environment?](#what)
- [How do I create one?](#create)
- [How does it work?](#how)
- [Where do they live?](#where)
- [Related tools](#related)
- [Tips](#tips)
- [Reference](#reference)

???

TODO

- TOC/Outline
- Deploy to GitHub
- Run commands on Ubuntu?
- Better [typography](https://fonts.google.com/?category=Monospace)
- https://github.com/gnab/remark/wiki/Configuration#highlighting
- Slide timing
- Personal links


---
name: default
layout: true

<!-- TODO: TOC link -->

---
name: assumptions
template: default

## Assumptions

You've got Python 3 installed (though much of this applies to Python 2)

You're not using Python on Windows (but references provide instructions)

You're somewhat familiar with a Unix-like command line

You've used `pip install`

???

- Installing Python 3 is non-trivial. Refer to "Installing" slide.

---
name: what
template: default

## What is a virtual environment?

An isolated workspace for Python projects

A filesystem directory containing a `python` executable and user-installed packages, that can be used independently of other Python environments

"other Python environments"

- OS-installed Python, aka "system Python"
- User-installed Python, like Anaconda
- Other virtual environments 

"package" == "[distribution package](https://packaging.python.org/glossary/#term-distribution-package)"

- Includes libraries, frameworks, standalone apps, etc.
- Usually installed via `pip install`
- Installed to `site-packages` directory relative to `sys.prefix`

???

- These are my definitions, inspired by others
- `sys.prefix` == "where `python` thinks it's installed"

---
template: default

### What problems does it solve?

Conflicting package dependencies between projects

Using different versions of Python for different projects

Keeping system Python clean (especially on shared hosts)

---
name: create
template: default

## How do I create one?

---
template: default

### Before 

```
$ for cmd in python pip; do which $cmd && $cmd --version; done
/usr/bin/python
Python 2.7.10

$ for cmd in python3 pip3; do which $cmd && $cmd --version; done
/usr/local/bin/python3
Python 3.6.2
/usr/local/bin/pip3
pip 9.0.1 from /usr/local/lib/python3.6/site-packages (python 3.6)

$ python3 -c "import sys; print(sys.prefix)"
/usr/local/Cellar/python3/3.6.2/Frameworks/Python.framework/Versions/3.6

$ pip3 list
pip (9.0.1)
setuptools (32.2.0)
wheel (0.29.0)
```

???

- `pip3` vs `pip` vs `python3 -m pip`
- macOS comes with Python 2, but not `pip`
- Used Homebrew to install Python 3, which uses symlinks to install directories


TODO

- `tree` of `sys.prefix`?

---
template: default

### Python 3 makes creation easy

```
$ python3 -m venv api_venv

$ source api_venv/bin/activate

(api_venv) $ pip install requests
# ... install output ...

(api_venv) $ python
Python 3.6.2 (default, Jul 17 2017, 16:44:45)
>>> import requests
# Play with a web API...
>>> ^D

(api_venv) $ deactivate

$ exit
```

This is a minimal example, *not* a best practice.

???

- Note that I used `python`, not `python3`

---
template: default

### After

```
(api_venv) $ for cmd in python pip; do which $cmd && $cmd --version; done
/Users/brian/Code/api_venv/bin/python
Python 3.6.2
/Users/brian/Code/api_venv/bin/pip
pip 9.0.1 from /Users/brian/Code/api_venv/lib/python3.6/site-packages (python 3.6)

(api_venv) $ python -c "import sys; print(sys.prefix)"
/Users/brian/Code/api_venv

(api_venv) $ pip list
certifi (2017.7.27.1)
chardet (3.0.4)
idna (2.5)
pip (9.0.1)
requests (2.18.3)
setuptools (28.8.0)
urllib3 (1.22)
```

---
name: how
template: default

## How does it work?

Let's go back to where it all started...

---
name: venv-2
template: default
layout: true

### Virtual environments in Python 2

---
template: venv-2

```
$ pip2 install virtualenv
# ... install output ...

$ pip2 list
pip (9.0.1)
setuptools (32.1.0)
virtualenv (15.1.0)
wheel (0.29.0)
```

Check your OS package manager. On Ubuntu, use `apt install virtualenv`.

???

- First version of `virtualenv` on PyPI on 9/14/2007
- Using `pip2` to guarantee Python 2
- Latest version of Python 2 installed via Homebrew

```
$ for cmd in python2 pip2; do which $cmd && $cmd --version; done
/usr/local/bin/python2
Python 2.7.13
/usr/local/bin/pip2
pip 9.0.1 from /usr/local/lib/python2.7/site-packages (python 2.7)
```

---
template: venv-2

```
$ virtualenv api_virtualenv
New python executable in /Users/brian/Code/api_virtualenv/bin/python2.7
Also creating executable in /Users/brian/Code/api_virtualenv/bin/python
Installing setuptools, pip, wheel...done.
```

Copies Python binary

Tells copied Python binary to use isolated `sys.prefix`

- Creates hacked copy of `site.py`, plus symbolic links to required modules
- But, Python upgrades or OS could cause problems by changing `site.py`

Installs `pip` and `setuptools`

Adds `activate` scripts and `python` symbolic links

???

- Latest version of `virtualenv` is from 11/16/2016

---
template: venv-2

```
$ tree -L 4 --dirsfirst -I '*.pyc|*.dist-info' api_virtualenv
api_virtualenv
├── bin
│   ├── activate
│   # more activate scripts
│   ├── easy_install
│   ├── easy_install-2.7
│   ├── pip
│   ├── pip2
│   ├── pip2.7
│   ├── python -> python2.7
│   ├── python-config
│   ├── python2 -> python2.7
│   ├── python2.7
│   └── wheel
├── include
│   └── python2.7 -> /usr/local/Cellar/python/2.7.13_1/...
```
---
template: venv-2
```
├── lib
│   └── python2.7
│       ├── config -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── distutils
│       │   ├── __init__.py
│       │   └── distutils.cfg
│       ├── encodings -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── lib-dynload -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── site-packages
│       │   ├── pip
│       │   ├── pkg_resources
│       │   ├── setuptools
│       │   ├── wheel
│       │   └── easy_install.py
```
---
template: venv-2
```
│       ├── no-global-site-packages.txt
│       ├── orig-prefix.txt
│       ├── site.py
│       ├── abc.py -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── codecs.py -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── fnmatch.py -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── genericpath.py -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── locale.py -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── os.py -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── posixpath.py -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── re.py -> /usr/local/Cellar/python/2.7.13_1/...
│       # more symbolic links
└── pip-selfcheck.json

14 directories, 42 files
```

???

- I haven't installed any packages!

TODO

- What do the the non-`.py` files do?

---
name: activate
template: default

### What does `activate` do?

```
$ source api_virtualenv/bin/activate

(api_virtualenv) $ which python && python --version
/Users/brian/Code/api_virtualenv/bin/python
Python 2.7.13

(api_virtualenv) $ echo $PATH
/Users/brian/Code/api_virtualenv/bin:...:/usr/local/bin:/usr/bin:...

(api_virtualenv) $ echo $VIRTUAL_ENV
/Users/brian/Code/api_virtualenv
```

Easy access to `python` via `api_virtualenv/bin` as first directory in `$PATH`

Also defines `deactivate`, `$VIRTUAL_ENV`, and modifies `$PS1`

Use the virtual environment without `activate` via `api_virtualenv/bin/python`

---
name: venv-3
template: default
layout: true

### Virtual environments in Python 3

---
template: venv-3

Back to the present...

---
template: venv-3

Part of the language since Python 3.3

Creates symbolic link to Python binary, instead of a copy

Adds `pyvenv.cfg` to tell Python that this is a virtual environment

No need for hacked `site.py` and module symbolic links

`activate` does pretty much the same thing

```
$ cat api_venv/pyvenv.cfg
home = /usr/local/bin
include-system-site-packages = false
version = 3.6.2
```

???

Python 3.3 released on 9/29/2012

---
template: venv-3

```
$ tree -L 4 --dirsfirst -I '__pycache__|*.dist-info' api_venv
api_venv
├── bin
│   ├── activate
│   # more activate scripts
│   ├── chardetect
│   ├── easy_install
│   ├── easy_install-3.6
│   ├── pip
│   ├── pip3
│   ├── pip3.6
│   ├── python -> python3
│   └── python3 -> /usr/local/bin/python3
├── include
```
---
```
├── lib
│   └── python3.6
│       └── site-packages
│           ├── certifi
│           ├── chardet
│           ├── idna
│           ├── pip
│           ├── pkg_resources
│           ├── requests
│           ├── setuptools
│           ├── urllib3
│           └── easy_install.py
├── pip-selfcheck.json
└── pyvenv.cfg

13 directories, 14 files
```

???

- Much shorter, even with installed packages!

---
name: where
template: default

## Where do they live?

It depends on the project, and your preferences...

---
template: where

`~/Code/project/`

- `source bin/activate`
- `(project)` in prompt
- Allows for projects with same name
- Need `bin`, etc. in `.gitignore`
- `bin`, etc. could conflict with your project
- Hard to delete

---
template: where

`~/Code/project/venv/`

- `source venv/bin/activate`
- `(venv)` in prompt
- Allows for projects with same name
- Easy to delete
- Common pattern, similar to Node and Ruby
- Need `venv` in `.gitignore`

---
template: where

`~/.virtualenvs/project/`

- `source ~/.virtualenvs/project/bin/activate`
- No need to `.gitignore`
- `(project)` in prompt
- Easy to delete
- Consistent (i.e., aliasable) `activate`
- Project names must be unique

???

This is my preference

---
name: related
template: default

## Related tools

- virtualenvwrapper
- pipsi (e.g. http-prompt), vs. manual
- Environment variables (e.g., autoenv, direnv)
- pew
- pyenv and pyenv-virtualenv
- pythonz
- tox

???

TODO

- Bullets and/or slides other tools

---
name: wrapper
template: default
layout: true

### virtualenvwrapper

---
template: wrapper

```
$ pip2 install virtualenvwrapper
$ cat >> ~/.bash_profile 
export WORKON_HOME=$HOME/.virtualenvs
export PROJECT_HOME=$HOME/Code
export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python2
source /usr/local/bin/virtualenvwrapper.sh
^D
```

Adds shell commands for adding and activating virtual environments by name

Creates virtual environments in `$WORKON_HOME`

Creates project directories in `$PROJECT_HOME`

Adds hook scripts for `activate` and `deactivate` (e.g., for environment variables)

---
template: wrapper

```
$ mkproject -p python3 api_project
# ...
New python executable in /Users/brian/.virtualenvs/api_project/bin/python3.6
# ...
Setting project for api_project to /Users/brian/Code/api_project

(api_project) $ echo $VIRTUAL_ENV
/Users/brian/.virtualenvs/api_project

(api_project) $ deactivate

$ cd /path/to/some/other/place

$ workon api_project

(api_project) $ pwd
/Users/brian/Code/api_project
```

---
template: wrapper

```
(api_project) $ tree -L 4 --dirsfirst -I '__pycache__|*.dist-info' $VIRTUAL_ENV
/Users/brian/.virtualenvs/api_project
├── bin
│   ├── activate
│   ├── postactivate
│   ├── postdeactivate
│   ├── preactivate
│   ├── predeactivate
│   ├── python -> python3.6
│   ├── python3.6
# snip
├── lib
│   └── python3.6
│       ├── collections -> /usr/local/Cellar/python3/3.6.2/...
```

---
template: wrapper

Uses `virtualenv` and its hacks

Slows down shell startup (but has a lazy-loading option)

Some attempts at a `venv`-based alternative, but nothing as prevelant

IMHO:

Better to use `venv` and `virtualenv` directly, and set up your own shortcuts

See my [`.bash_profile`](https://github.com/bhrutledge/dotfiles/blob/2b9e35ff6efe3d000dbed6f6b50e04599c49f0a5/.bash_profile) and [`.bashrc`](https://github.com/bhrutledge/dotfiles/blob/2b9e35ff6efe3d000dbed6f6b50e04599c49f0a5/.bashrc#L22-L63) for an example

---
name: tips
template: default

## Tips

Prefer `venv` over `virtualenv` for Python 3

Treat venvs as ephemeral

- Don't manually edit files in the venv
- Don't try to relocate venvs; recreate them instead
- Upgrade Python using a new venv
- Delete venvs with `rm -rf`

The `pyvenv` command was deprecated in favor of `python3 -m venv`

Outside of a venv, be explicit about which Python you're using:

```
$ python2.7 -m virtualenv
$ python3.6 -m pip
```

Use [`pip install -r requirements.txt`](https://pip.pypa.io/en/stable/user_guide/#requirements-files), and check out [pip-tools](https://github.com/jazzband/pip-tools)

???

- Think twice before using `pip` outside of a venv

---
name: reference
template: default

## Reference

- [Python Virtual Environments - a primer](https://realpython.com/blog/python/python-virtual-environments-a-primer/#disqus_thread)
- [Don't Make Us Say We Told You So: virtualenv for New Pythonistas](https://www.youtube.com/watch?v=Xdv7vwIIThY)
- [Reverse-engineering Ian Bicking's brain: inside pip and virtualenv](http://pyvideo.org/pycon-us-2011/pycon-2011--reverse-engineering-ian-bicking--39-s.html)
    - [Slides](https://carljm.github.io/pipvirtualenv-preso/)
    - Watch to the end to see the genesis of `venv` in Python 3
- [venv documentation](https://docs.python.org/3/library/venv.html)
- [PEP 405 -- Python Virtual Environments](https://www.python.org/dev/peps/pep-0405/)
- [virtualenv documentation](https://virtualenv.pypa.io/en/stable/)

???

TODO

- https://docs.python.org/3/library/sys.html
- https://docs.python.org/3/library/site.html
- https://github.com/python/cpython/tree/master/Lib/venv
- [Installing Python Modules](https://docs.python.org/3/installing/index.html)
- Evolution of `virtualenv` to `venv`
- [Virtualenv's `bin/activate` is Doing It Wrong](https://gist.github.com/datagrok/2199506)

---
name: dev
template: default

## Using with other dev tools

- IDLE
    - `python3 -m idlelib`
    - `python2 -c "from idlelib.PyShell import main; main();`
- tmux
- Fabric
- jupyter
- Apache
- vim
- jedi

---
name: alternatives
template: default

## Alternatives

[Pipenv: Sacred Marriage of Pipfile, Pip, & Virtualenv](http://docs.pipenv.org/en/latest/)

- "[`Pipfile` will be superior to `requirements.txt`](https://github.com/pypa/pipfile)"
- Relies on [pew](https://github.com/berdario/pew), which only uses `virtualenv`
- [Default setup adds ugly hash to venv name](https://github.com/kennethreitz/pipenv/pull/238)

[Conda](https://conda.io/docs/index.html)

- Used by, but not exclusive to, the Anaconda distribution of Python
- Handles package installation and virtual environments for multiple languages

[Docker](https://www.fullstackpython.com/docker.html)

- Use a `Dockerfile` to build an image for a specific Python version with a specific set of packages
- Run scripts, apps, etc. in containers using that image

---
name: installing
template: default

## Installing/upgrading Python 3

- [Python Setup and Usage](https://docs.python.org/3/using/index.html)
- [Properly Installing Python](https://python-guide-pencheil.readthedocs.io/en/latest/starting/installation.html)
- [pyenv: Simple Python version management](https://github.com/pyenv/pyenv)
- [Anaconda](https://www.continuum.io/downloads)

Ubuntu

- `apt install python3-venv python3.6-venv`
- Don't try to change `/usr/bin/python3`, e.g. via `update-alternatives`
    - [`ModuleNotFoundError: No module named 'apt_pkg'`](https://bugs.launchpad.net/ubuntu/+source/python3.6/+bug/1631367)
- Use `python3.6 -m venv` instead

???

- Mention Vagrantfile in repo

TODO

- TL;DR version
