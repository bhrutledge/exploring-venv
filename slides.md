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

Say "venv" and "virtualenv" instead of "virtual environment"

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

You've got Python 3 installed on macOS or Linux

You're somewhat familiar with a Unix-like command line

You've used `pip install`

???

- Before we dive in, a few assumptions I've made in putting together these slides
- Doesn't mean this won't be useful, I just won't be covering how to get to this point
- I'm also going to talk about Python 2
- I'm not a Windows user, but the references provide instructions
- Installing Python is non-trivial; we could do a whole talk (*hint*). See "Installing" slide.
- Show of hands

---
name: what

## What are they?

Isolated workspaces for Python projects

???

- High level, I like to think of them like this
- When I start working on a new project, I usually create a venv

--

A place to install and use packages independently from other Python environments

???

- More concretely
- Let's define a couple things

--

"packages" == "[distribution packages](https://packaging.python.org/glossary/#term-distribution-package)"

- Libraries, frameworks, standalone apps, etc.
- Installed with `pip` into `site-packages` directory

???

- Python Packaging Authority
- requests, pandas, Django, Flask, jupyter, Scrapy
- `site-packages` inside Python environment

--

"other Python environments"

- OS-installed Python, aka "system Python"
- User-installed Python, e.g. from python.org or Anaconda
- Other virtual environments 

???

- An installation of Python

---

### Why should I use them?

Avoid conflicting package dependencies

???

- At work, we use Django 1.8, my band's website uses 1.11
- Allows me to develop both on the same laptop
- System Python might rely on an older version of a library, and you want to use the newer one
- Don't touch system Python

--

Use different versions of Python for different projects

???

- At work, we use Python 2.7, my band's website uses Python 3.6
- venvs are tied to a specific Python executable, i.e. version

--

Install packages without `sudo pip install`

???

- Might see instructions online for installing to system Python
- But we don't want to touch system Python
- Can't use `sudo` on shared hosting
- venvs usually live in your home directory
- Install whatever you want

---
name: create

## How do I create one?

A minimal example:

???

- Python 3 makes this easy
- Not a best practice
- Let's say I want to play with a web API. I'll want to use `requests`.

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

???

- Now I'm inside the venv, and it's safe to install packages

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

???

- Now I'm outside the venv

---

### What just happened?

???

- Let's look at our shell environment before and after we ran `activate`

---

### Before `activate` 

```
$ which python3 && python3 --version
/usr/local/bin/python3
Python 3.6.2
```

???

- Installed via Homebrew

--

```
$ pip3 list
pip (9.0.1)
setuptools (32.2.0)
wheel (0.29.0)
```

???

- Using `pip3` to install Python 3 packages

--

```
$ which python && python --version
/usr/bin/python
Python 2.7.10
```

???

- What about just `python`?

---

### After `activate`

```
(api_venv)$ which python && python --version
/Users/brian/Code/api_venv/bin/python
Python 3.6.2
```
???

- `venv` lets us use `python` instead of `python3`

--

```
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

- Clearly different environments for running Python

---
name: how

## How do they work?

Let's go back in time, and into the weeds...

---
layout: true

### Python 2: `virtualenv`

---

```
$ pip2 install virtualenv
```

???

- First version of `virtualenv` on PyPI on 9/14/2007
- Using `pip2` to guarantee Python 2
- Latest version of Python 2 installed via Homebrew, includes pip
- On Ubuntu, use `apt install virtualenv`

--

```
$ python2 -m virtualenv api_virtualenv
New python executable in /Users/brian/Code/api_virtualenv/bin/python2.7
Also creating executable in /Users/brian/Code/api_virtualenv/bin/python
Installing setuptools, pip, wheel...done.
```

???

- Similar to `python3 -m venv`
- Output is informative, unlike `venv`

--

Or

```
$ virtualenv api_virtualenv
```

???

- Commonly documented

--

But it's not clear which version of Python we're using

???

- Specifying a different version requires a command line argument

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

- Let's see what's inside our virtualenv
- Abbreviated output of `tree -L 4 --dirsfirst -I '*.pyc|*.dist-info' api_virtualenv`
- All this, and I haven't installed any packages!
- Isolated `site-packages` is what we want; that's where packages go
- Adds `activate` scripts
- Handy `python` symbolic link
- What does all the other stuff in `lib` do?

---

Tells Python to use the virtualenv directory as [`sys.prefix`](https://docs.python.org/2/library/sys.html#sys.prefix)

```
$ /usr/local/bin/python2 -c "import sys; print(sys.prefix)"
/usr/local/Cellar/python/2.7.13_1/Frameworks/Python.framework/Versions/2.7

$ api_virtualenv/bin/python -c "import sys; print(sys.prefix)"
/Users/brian/Code/api_virtualenv/bin/..
```

???

- `sys.prefix` == "directory where Python thinks it's installed"
- Determined by moving up from location of executable and looking for specific files
- Homebrew install creates symbolic links to executables
- https://carljm.github.io/pipvirtualenv-preso/#10
- But now we don't have a standard library; batteries not included

--

Creates hacked copy of [`site.py`](https://docs.python.org/2/library/site.html) to use standard library from original `sys.prefix`

```
$ cat api_virtualenv/lib/python2.7/orig-prefix.txt
/usr/local/Cellar/python/2.7.13_1/Frameworks/Python.framework/Versions/2.7
```

???

- `site.py` tells Python where to look for packages
- Load the standard library from over here
- https://carljm.github.io/pipvirtualenv-preso/#20

--

Creates symbolic links to standard library modules required by `site.py`

???

- I know we're in the weeds, but I think this is really interesting
- A crucial part of developing Python is a pretty big hack

--

Python upgrades or OS could cause problems by changing `site.py`

???

- Hacks are brittle
- Last release of `virtualenv` on 11/16/2016, 3.6 released 12/23/2016


---
layout: true

### Python 3: `venv`

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

- Back to the venv that we created earlier
- `tree -L 4 --dirsfirst -I '__pycache__|*.dist-info' api_venv`
- Much shorter, even with installed packages!
- Still have `site-packages`, but no symbolic links
- Creates symbolic link to Python binary, instead of a copy

---

`venv` added to standard library in Python 3.3

???

- Python 3.3 released on 9/29/2012

--

No need for hacked `site.py` and standard library symbolic links

Adds `pyvenv.cfg` to tell Python that this is a venv

```
$ cat api_venv/pyvenv.cfg
home = /usr/local/bin
include-system-site-packages = false
version = 3.6.2
```

--

`pyvenv` command deprecated; use `python3 -m venv`

???

TODO

- Customizable

---
layout: false

### What does `activate` do?

```
$ source api_virtualenv/bin/activate
```

???

- `source` allows script to modify current shell session
- Doesn't work without it

--

```
(api_virtualenv)$ echo $VIRTUAL_ENV
/Users/brian/Code/api_virtualenv
```

???

- Modifies `$PS1` using directory name
- `$VIRTUAL_ENV` is root of virtualenv

--

```
(api_virtualenv)$ which python && python --version
/Users/brian/Code/api_virtualenv/bin/python
Python 2.7.13
```

--

```
(api_virtualenv)$ echo $PATH
/Users/brian/Code/api_virtualenv/bin:...:/usr/local/bin:/usr/bin:...
```

???

- Modifies `$PATH`, defines `deactivate`
- Before `activate` and after `deactivate` wouldn't have `api_virtualenv/bin`

--

Just a shortcut to `python`

???

- Pretty much the same in Python 2 and 3

--

Use the virtualenv without `activate` via `api_virtualenv/bin/python`

???

TODO

- Rerun in `api_venv`
- [Virtualenv's `bin/activate` is Doing It Wrong](https://gist.github.com/datagrok/2199506)
    - Shell-specific scripts
    - Brittle `$PS1` modification
    - Doesn't solve setting/unsetting environment variables
    - Sub-shell is better


---
name: where
layout: false

## Where do they live?

Depends on the project, and your preferences

Should be isolated from your source code

???

### `~/Code/project/`

```
$ python3 -m venv project
$ cd project
$ source bin/activate
(project)$ vim project.py
```

Need `bin`, etc. in `.gitignore`

`bin`, etc. could conflict with your project

Hard to delete

Not recommended

---

### `~/venvs/project/`

```
$ python3 -m venv ~/venvs/project
$ source ~/venvs/project/bin/activate
(project)$ cd ~/Code/project
(project)$ vim project.py
```

--

Completely isolated from code

Consistent (i.e., aliasable) `activate`

???

No need to `.gitignore`

Replace `~/venvs` with whatever you want

--

This is my preference

---

### `~/Code/project/venv/`

```
$ cd ~/Code/project
$ python3 -m venv venv
$ source venv/bin/activate
(venv)$ vim project.py
```

--

Common pattern, similar to Node and Ruby

Need `venv` in `.gitignore`

Use `--prompt` option for `(project)$`

???

- Don't put venv in source control

---
name: related

## Related tools

???

- Make venv easier to work with
- Reduce typing, establish conventions

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
# Then add some stuff to ~/.bash_profile, and start a new shell
```

--

```
$ mkproject -p python3 api_project
```

???

- Shell commands replace `python3 -m venv`

--

```
(api_project)$ pwd && echo $VIRTUAL_ENV
/Users/brian/Code/api_project
/Users/brian/.virtualenvs/api_project
```

???

- Created and `cd`'d into directory for us

--

```
(api_project)$ deactivate
```

--

```
$ cd ~
```

--

```
$ workon api_project
```

--

```
(api_project)$ pwd
/Users/brian/Code/api_project
```

---

Convenient access to virtualenvs by name

Keeps virtualenvs and project directories in consistent, isolated locations

Adds pre- and post- hooks for `activate` and `deactivate`

???

- Set/unset environment variables

--

Adds hacks on top of `virtualenv` and its hacks

Slows down shell startup

???

- Has a lazy-loading option
- Some attempts at a `venv`-based alternative, but nothing as prevelant

--

IMHO, better to use `venv` / `virtualenv` directly, and set up your own shortcuts

See my [`.bash_profile`](https://github.com/bhrutledge/dotfiles/blob/2b9e35ff6efe3d000dbed6f6b50e04599c49f0a5/.bash_profile)
and [`.bashrc`](https://github.com/bhrutledge/dotfiles/blob/2b9e35ff6efe3d000dbed6f6b50e04599c49f0a5/.bashrc#L22-L63)
for an example

???

- I used virtualenvwrapper for a long time

---
name: tips
layout: false

## Tips

--

Treat venvs as ephemeral

- Don't manually edit files in the venv
- Don't try to relocate venvs; recreate them instead
- Upgrade Python using a new venv
- Delete with `rm -rf`

???

- `virtualenvwrapper` encourages editing venv scripts

--

Prefer `venv` over `virtualenv` for Python 3

--

Outside of a venv, be explicit about which Python you're using:

```
$ python2.7 -m virtualenv
$ python3.6 -m pip
```

???

- Think twice before using `pip` outside of a venv

--

Use [`pip install -r requirements.txt`](https://pip.pypa.io/en/stable/user_guide/#requirements-files), and check out [pip-tools](https://github.com/jazzband/pip-tools)

???

TODO

- `#!/usr/bin/env python3` in scripts
- Don't need venv for scripts that only use standard library
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

### Using with other dev tools

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

### Alternatives

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

### Installing/upgrading Python 3

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

TODO

- Vagrantfile in repo
- TL;DR version
