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

This is the first try, the `soundsgood.__init__.py` looks like this.

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

# Try 2:
