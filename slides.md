# Exploring <br> Virtual Environments

## with Brian Rutledge

- [What are they?](#what)
- [How do I create one?](#create)
- [How do they work?](#how)
- [Where do they live?](#where)
- [Related tools](#related)
- [Tips](#tips)
- [Reference](#reference)

???

TODO

- TOC/Outline
- `python2 -m pip install --user`
- Run commands on Ubuntu?
- Better typography
    - https://fonts.google.com/?category=Monospace
    - Quotes
- https://github.com/gnab/remark/wiki/Configuration#highlighting
- Slide timing
- Personal links

---
name: assumptions

## Assumptions

You've got Python 3 installed

You're not using Python on Windows

You're somewhat familiar with a Unix-like command line

You've used `pip install`

???

- Before we dive in...
- Much of this applies to Python 2
- I'm not a Windows user, but the references provide instructions
- Installing Python and pip is non-trivial; we could do a whole talk. See "Installing" slide.
- Show of hands

---
name: what

## What are they?

--

Isolated workspaces for Python projects

???

- When I start working on a new project, I usually create a venv

TODO

- Except for scripts that only use standard library

--

A place to install and use packages independently from other Python environments

--

"packages" == "[distribution packages](https://packaging.python.org/glossary/#term-distribution-package)"

- Libraries, frameworks, standalone apps, etc.
- Installed with `pip` into Python's `site-packages` directory

???

- Python Packaging Authority
- requests, pandas, Django, Flask, jupyter, Scrapy

--

"other Python environments"

- OS-installed Python, aka "system Python"
- User-installed Python, e.g. from python.org or Anaconda
- Other virtual environments 

---

### Why should I use them?

--

Avoid conflicting package dependencies between projects

???

- Django 1.8 for work vs. 1.11 for band's website
- Allows me to develop both on the same laptop

--

Use different versions of Python for different projects

???

- Python 2.7 for work vs. Python 3.6 for band's website

--

Install packages without `sudo pip install`

???

- Might see instructions online for installing to system Python
- Can't use `sudo` on shared hosting
- venvs usually live in your home directory
- Install whatever you want

---
name: create

## How do I create one?

A minimal example:

???

- Let's say I want to play with a web API
- Python 3 makes this easy
- Not a best practice

--

```
$ python3 -m venv api_venv
```

???

- Run the `venv` module as a script
- We'll see the contents of the directory later

--

```
$ source api_venv/bin/activate
```

???

- We'll see what this does later

--

```
(api_venv)$ pip install requests
# ... install output ...
```

--

```
(api_venv)$ python
Python 3.6.2 (default, Jul 17 2017, 16:44:45)
>>> import requests
# Play with a web API...
>>> ^D
```

???

- We'll see why `python` works later

--

```
(api_venv)$ deactivate
```

--

```
$ 
```

---

### What just happened?

???

- Let's look at our environment before and after we ran `activate`

---

### Before `activate` 

```
$ which python3 && python3 --version
/usr/local/bin/python3
Python 3.6.2

$ python3 -c "import sys; print(sys.prefix)"
/usr/local/Cellar/python3/3.6.2/Frameworks/Python.framework/Versions/3.6

$ pip3 list
pip (9.0.1)
setuptools (32.2.0)
wheel (0.29.0)
```

???

- `sys.prefix` == "directory where Python thinks it's installed"
- `sys.prefix` is location of Homebrew installation, which creates symbolic links to executables
- Otherwise, `sys.prefix` would be `/usr/local`
- `pip3` vs `pip`

--

```
$ which python && python --version
/usr/bin/python
Python 2.7.10
```

---

### After `activate`

```
(api_venv)$ which python && python --version
/Users/brian/Code/api_venv/bin/python
Python 3.6.2

(api_venv)$ python -c "import sys; print(sys.prefix)"
/Users/brian/Code/api_venv

(api_venv)$ pip list
certifi (2017.7.27.1)
chardet (3.0.4)
idna (2.5)
pip (9.0.1)
requests (2.18.3)
setuptools (28.8.0)
urllib3 (1.22)
```

???

- Now I'm using `python`, not `python3`
- `sys.prefix` is the directory created by `venv`
- Clearly different environments for running Python

---
name: how

## How do they work?

Let's go back in time...

---
layout: true

### Virtual environments in Python 2

---

```
$ pip2 install virtualenv
```

???

- First version of `virtualenv` on PyPI on 9/14/2007
- Using `pip2` to guarantee Python 2
- Latest version of Python 2 installed via Homebrew, includes pip

TODO

- On Ubuntu, use `apt install virtualenv`

--

```
$ python2 -m virtualenv api_virtualenv
New python executable in /Users/brian/Code/api_virtualenv/bin/python2.7
Also creating executable in /Users/brian/Code/api_virtualenv/bin/python
Installing setuptools, pip, wheel...done.
```

???

- Output is informative, unlike `venv`

--

Or

```
$ virtualenv api_virtualenv
```

But it's not clear which version of Python we're using

???

- Most common instruction

---
classes: small

.small[
```
api_virtualenv
├── bin
│   ├── activate
│   ├── pip
│   ├── pip2.7
│   ├── python -> python2.7
│   ├── python2.7
│   # more executables and activate scripts
├── include
│   └── python2.7 -> /usr/local/Cellar/python/2.7.13_1/...
├── lib
│   └── python2.7
│       ├── distutils
│       ├── site-packages
│       │   ├── pip
│       │   ├── setuptools
│       │   ├── wheel
│       │   # more packaging resources
│       ├── no-global-site-packages.txt
│       ├── orig-prefix.txt
│       ├── site.py
│       ├── os.py -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── posixpath.py -> /usr/local/Cellar/python/2.7.13_1/...
│       ├── re.py -> /usr/local/Cellar/python/2.7.13_1/...
│       # many more symbolic links
```
]

???

- Let's see what's inside
- Abbreviated output of `tree -L 4 --dirsfirst -I '*.pyc|*.dist-info' api_virtualenv`
- Adds `activate` scripts
- `python` symbolic link is why `python` command works
- `include` contains C headers for compiled packages
- There's `site-packages`
- All this, and I haven't installed any packages!
- What does all the stuff in `lib` do?

---

Tells Python that [`sys.prefix`](https://docs.python.org/2/library/sys.html#sys.prefix) is the virtualenv directory

```
$ api_virtualenv/bin/python -c "import sys; print(sys.prefix)"
/Users/brian/Code/api_virtualenv/bin/..
```

???

- Recall:
    - Where Python thinks it's installed, and looks for its standard library
    - pip installs packages to `site-packages`
- I'm using the virtualenv without `activate`
- But now we don't have a standard library

--

Creates hacked copy of [`site.py`](https://docs.python.org/2/library/site.html) to use standard library from `orig-prefix.txt`

```
$ cat orig-prefix.txt
/usr/local/Cellar/python/2.7.13_1/Frameworks/Python.framework/Versions/2.7
```

???

- `site.py` tells Python where to look for packages
- https://carljm.github.io/pipvirtualenv-preso/#20

--

Creates symbolic links to standard library modules required by `site.py`

--

Python upgrades or OS could cause problems by changing `site.py`

???

- Last release of `virtualenv` on 11/16/2016

---
layout: false

### What does `activate` do?

```
$ source api_virtualenv/bin/activate

(api_virtualenv)$ echo $VIRTUAL_ENV
/Users/brian/Code/api_virtualenv
```

???

- `source` allows script to modify current shell session
- Modifies `$PS1` using directory name

--

```
(api_virtualenv)$ which python && python --version
/Users/brian/Code/api_virtualenv/bin/python
Python 2.7.13

(api_virtualenv)$ echo $PATH
/Users/brian/Code/api_virtualenv/bin:...:/usr/local/bin:/usr/bin:...
```

--

Just a shortcut to `python`

???

- Use the virtualenv without `activate` via `api_virtualenv/bin/python`
- Also defines `deactivate` 
- [Virtualenv's `bin/activate` is Doing It Wrong](https://gist.github.com/datagrok/2199506)
    - Shell-specific scripts
    - Brittle `$PS1` modification
    - Doesn't solve setting/unsetting environment variables
    - Sub-shell is better


---
layout: true

### Virtual environments in Python 3

---

.small[
```
api_venv
├── bin
│   ├── activate
│   ├── pip
│   ├── pip3.6
│   ├── python -> python3
│   ├── python3 -> /usr/local/bin/python3
│   # more executables and activate scripts
├── include
├── lib
│   └── python3.6
│       └── site-packages
│           ├── certifi
│           ├── chardet
│           ├── idna
│           ├── pip
│           ├── requests
│           ├── setuptools
│           ├── urllib3
│           # more packaging resources
└── pyvenv.cfg
```
]

???

- `tree -L 4 --dirsfirst -I '__pycache__|*.dist-info' api_venv`
- Creates symbolic link to Python binary, instead of a copy
- Much shorter, even with installed packages!

---

Part of the language since Python 3.3

No need for hacked `site.py` and standard library symbolic links

Adds `pyvenv.cfg` to tell Python that this is a venv

```
$ cat api_venv/pyvenv.cfg
home = /usr/local/bin
include-system-site-packages = false
version = 3.6.2
```

???

- Python 3.3 released on 9/29/2012

--

`activate` does pretty much the same thing

`pyvenv` command deprecated in favor of `python3 -m venv`

???

TODO

- Customizable

---
name: where
layout: false

## Where do they live?

It depends on the project, and your preferences...

---

### `~/Code/project/`

```
$ python3 -m venv project
$ cd project
$ source bin/activate
(project)$ vim project.py
```

--

Need `bin`, etc. in `.gitignore`

`bin`, etc. could conflict with your project

Hard to delete

???

- Flattest, easy to type

--

Not recommended

---

### `~/Code/project/venv/`

```
$ cd project
$ python3 -m venv project venv
$ source venv/bin/activate
(venv)$ vim project.py
```

--

Need `venv` in `.gitignore`

Easy to delete

Common pattern, similar to Node and Ruby

Use `--prompt` option for `(project)$`

---

### `~/.venvs/project/`

```
$ cd project
$ python3 -m venv ~/.venvs/project
$ source ~/.venvs/project/bin/activate
(project)$ vim project.py
```

--

No need to `.gitignore`

Easy to delete

Consistent (i.e., aliasable) `activate`

venv names must be unique

--

This is my preference

---
name: related

## Related tools

???

TODO

- TOC w/ links
- https://virtualenvwrapper.readthedocs.io/
- https://github.com/berdario/pew
- https://github.com/mitsuhiko/pipsi (vs. manual script install)
- https://github.com/pyenv/pyenv-virtualenv
- https://tox.readthedocs.io/en/latest/

---
layout: true

### virtualenvwrapper

---

```
$ pip2 install virtualenvwrapper
$ cat >> ~/.bash_profile 
export WORKON_HOME=$HOME/.virtualenvs
export PROJECT_HOME=$HOME/Code
export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python2
source /usr/local/bin/virtualenvwrapper.sh
^D
```

Adds shell commands for adding and activating virtualenvs by name

Creates virtualenvs in `$WORKON_HOME`

Creates project directories in `$PROJECT_HOME`

Adds hook scripts for `activate` and `deactivate` (e.g., for environment variables)

---

```
$ mkproject -p python3 api_project
# ...
New python executable in /Users/brian/.virtualenvs/api_project/bin/python3.6
# ...
Setting project for api_project to /Users/brian/Code/api_project

(api_project)$ echo $VIRTUAL_ENV
/Users/brian/.virtualenvs/api_project

(api_project)$ deactivate

$ cd /path/to/some/other/place

$ workon api_project

(api_project)$ pwd
/Users/brian/Code/api_project
```

---

.small[
```
/Users/brian/.virtualenvs/api_project
├── bin
│   ├── activate
│   ├── postactivate
│   ├── postdeactivate
│   ├── preactivate
│   ├── predeactivate
│   ├── python -> python3.6
│   ├── python3.6
│   # more executables and activate scripts
├── include
│   └── python3.6m -> /usr/local/Cellar/python3/3.6.2/...
├── lib
│   └── python3.6
│       # same stuff as Python 2 virtualenv
```
]

???

- `tree -L 4 --dirsfirst -I '__pycache__|*.dist-info' $VIRTUAL_ENV`

---

Uses `virtualenv` and its hacks

Slows down shell startup (but has a lazy-loading option)

Some attempts at a `venv`-based alternative, but nothing as prevelant

--

*IMHO*

Better to use `venv` and `virtualenv` directly, and set up your own shortcuts

See my [`.bash_profile`](https://github.com/bhrutledge/dotfiles/blob/2b9e35ff6efe3d000dbed6f6b50e04599c49f0a5/.bash_profile)
and [`.bashrc`](https://github.com/bhrutledge/dotfiles/blob/2b9e35ff6efe3d000dbed6f6b50e04599c49f0a5/.bashrc#L22-L63)
for an example

---
name: tips
layout: false

## Tips

Treat venvs as ephemeral

- Don't manually edit files in the venv
- Don't try to relocate venvs; recreate them instead
- Upgrade Python using a new venv
- Delete venvs with `rm -rf`

Prefer `venv` over `virtualenv` for Python 3

Outside of a venv, be explicit about which Python you're using:

```
$ python2.7 -m virtualenv
$ python3.6 -m pip
```

Use [`pip install -r requirements.txt`](https://pip.pypa.io/en/stable/user_guide/#requirements-files), and check out [pip-tools](https://github.com/jazzband/pip-tools)

???

- Think twice before using `pip` outside of a venv

TODO

- `#!/usr/bin/env python` in scripts
- Environment variables (e.g., autoenv, direnv)

---
name: reference

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

- https://glyph.twistedmatrix.com/2016/08/python-packaging.html
- https://github.com/python/cpython/tree/master/Lib/venv
- [Installing Python Modules](https://docs.python.org/3/installing/index.html)
- https://groups.google.com/forum/#!forum/python-virtualenv
- Evolution of `virtualenv` to `venv`
- GitHub issues

---
name: dev

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
