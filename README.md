# soundsgood
Cos I don't think https://docs.python.org/3/tutorial/modules.html#packages is at all sound...


# Python Packaging Pandora

I couldn't get my nested `module -> submodule -> *.py` packaging working? 

After quite a painful hours of Google + Stackoverflowing, the results point to:


- https://stackoverflow.com/questions/33743880/what-does-from-future-import-absolute-import-actually-do
- https://stackoverflow.com/questions/63728242/importerror-cannot-import-name-unknown-location
- https://stackoverflow.com/questions/8953844/import-module-from-subfolder

- https://packaging.python.org/tutorials/packaging-projects/
- https://docs.python.org/3/reference/import.html#__path__
- https://docs.python.org/3/tutorial/modules.html#packages

Although the last link in the bullet above looks "sound", I've tried an hour worth of variation of how to write the `__init__.py` files and I'm giving up so I made this repo to replicate the tutorial and hopefully document what really should be in the `___init__.py` files from the https://docs.python.org/3/tutorial/modules.html#packages tutorial given the following file structure:

```
soundsgood/                     Top-level package
      __init__.py               Initialize the sound package
      formats/                  Subpackage for file format conversions
              __init__.py
              wavread.py
              wavwrite.py
              aiffread.py
              aiffwrite.py
              auread.py
              auwrite.py
              ...
      effects/                  Subpackage for sound effects
              __init__.py
              echo.py
              surround.py
              reverse.py
              ...
      filters/                  Subpackage for filters
              __init__.py
              equalizer.py
              vocoder.py
              karaoke.py
              ...
```

# Try 1: Testing `__init__.py` for `soundsgood.filters.karaoke`

This is the first try [`19d66c`](https://github.com/alvations/soundsgood/commit/19d66c45f22eac2300c8a0ade5ac674d343d2be1), the `soundsgood.__init__.py` looks like this.

```python
from soundsgood import filters
```

and `soundsgood/filters/__init__.py` looks like this:

```python
from soundsgood.filters import karaoke
```

And after running `

```
cd && yes | python3 -m pip uninstall soundsgood && cd /path/to/soundsgood/ && python3 setup.py install --force && cd 
python3 -c "import soundsgood"
```

[out]:

```
WARNING: Skipping soundsgood as it is not installed.

Finished processing dependencies for soundsgood==0.0.1

Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "<frozen zipimport>", line 259, in load_module
  File "/usr/local/lib/python3.9/site-packages/soundsgood-0.0.1-py3.9.egg/soundsgood/__init__.py", line 2, in <module>
ImportError: cannot import name 'filters' from 'soundsgood' (/usr/local/lib/python3.9/site-packages/soundsgood-0.0.1-py3.9.egg/soundsgood/__init__.py)
```

### And yes, as expected it's *kaputt*... 

# Try 2: Try simply importing `import soundsgood.filters` in top-level `__init__.py`

Undoing the last edit on the top-level `soundsgood/__init__.py` and simply doing ([0d78465](https://github.com/alvations/soundsgood/commit/0d784659f537d7905edb81056d2820ee880678aa)):

```python
import soundsgood.filters
```

And running `cd && yes | python3 -m pip uninstall soundsgood && cd /path/to/soundsgood/ && python3 setup.py install --force && cd  && python3 -c "import soundsgood"`

### Of course, it kaputt-ed again! 

[out]:

```
 % cd && yes | python3 -m pip uninstall soundsgood && cd git-stuff/soundsgood/ && python3 setup.py install --force && cd
python3 -c "import soundsgood"
Found existing installation: soundsgood 0.0.1
Uninstalling soundsgood-0.0.1:
  Would remove:
    /usr/local/lib/python3.9/site-packages/soundsgood-0.0.1-py3.9.egg
Proceed (y/n)?   Successfully uninstalled soundsgood-0.0.1

...

Processing soundsgood-0.0.1-py3.9.egg
Copying soundsgood-0.0.1-py3.9.egg to /usr/local/lib/python3.9/site-packages
Adding soundsgood 0.0.1 to easy-install.pth file

Installed /usr/local/lib/python3.9/site-packages/soundsgood-0.0.1-py3.9.egg
Processing dependencies for soundsgood==0.0.1
Finished processing dependencies for soundsgood==0.0.1
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "<frozen zipimport>", line 259, in load_module
  File "/usr/local/lib/python3.9/site-packages/soundsgood-0.0.1-py3.9.egg/soundsgood/__init__.py", line 2, in <module>
ModuleNotFoundError: No module named 'soundsgood.filters'
```

# Try 3: Use absolute imports + `setuptools`, forget about `distutils`

After some more digging, I found solution 3 from https://stackoverflow.com/a/28154841/610569 

In [f738af](https://github.com/alvations/soundsgood/commit/f738af3381a8136eb7d65331266fcca5f3120580), we're going back to the `__init__.py`, and we use absolute imports: 

`soundsgood/__init__.py`:

```python
from soundsgood import filters
```

`soundsgood/filters/__init__.py`:`

```python
from soundsgood.filters import *
```

**AND** in the setup.py, use `setuptools.find_packages()`:

```
from setuptools import setup, find_packages


setup(
  name = 'soundsgood',
  packages = find_packages(),
  version = '0.0.2',
  description = "Cos I don't think https://docs.python.org/3/tutorial/modules.html#packages is at all sound...",
  url = 'https://github.com/alvations/soundsgood',
)
```

Voila!! It worked!

[out]:

```
cd && yes | python3 -m pip uninstall soundsgood && cd git-stuff/soundsgood/ && python3 setup.py install --force && cd
python3 -c "import soundsgood; from soundsgood.filters.karaoke import teo_heng; teo_heng()"

Found existing installation: soundsgood 0.0.1
Uninstalling soundsgood-0.0.1:
  Would remove:
    /usr/local/lib/python3.9/site-packages/soundsgood-0.0.1-py3.9.egg
Proceed (y/n)?   Successfully uninstalled soundsgood-0.0.1


Processing soundsgood-0.0.2-py3.9.egg
Copying soundsgood-0.0.2-py3.9.egg to /usr/local/lib/python3.9/site-packages
Adding soundsgood 0.0.2 to easy-install.pth file

Installed /usr/local/lib/python3.9/site-packages/soundsgood-0.0.2-py3.9.egg
Processing dependencies for soundsgood==0.0.2
Finished processing dependencies for soundsgood==0.0.2


Teo Heng KTV is committed to provide the very budget platform for all karaoke lovers to enjoy the highest quality karaoke system. Our No Alchohol, No Smoking, No Frills policies are to promote healthy and green karaoke culture among every generation.
```
