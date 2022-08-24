# Python Packaging 101
A look at all the python packaging build backend options

## An Overview
Python packaging has undergone quite a few changes over the years, but slowly is becoming more standardised.
The following PEPs have helped.

- [PEP 517 – A build-system independent format for source trees](https://peps.python.org/pep-0517/)
- [PEP 518 – Specifying Minimum Build System Requirements for Python Projects](https://peps.python.org/pep-0518/)
- [PEP 660 – Editable installs for pyproject.toml based builds (wheel based)](https://peps.python.org/pep-0660/)

PEP 517 and 518 provide a way to allow different libraries to build packages, removing complete reliance on `setuptools`.
The addition of PEP 660 means that the `pyproject.toml` can now be the sole source of truth for packaging, no more need for `setup.py` and `setup.cfg`.

## Purpose
The purpose of this repository is to write a python library and package it using each of the following build backends.
-  [setuptools](https://setuptools.pypa.io/en/latest/)
-  [flit](https://flit.pypa.io/en/latest/index.html)
-  [poetry](https://python-poetry.org/docs/)
-  [hatch](https://hatch.pypa.io/latest/)

Compare the workflows and see which one I prefer.

## Writing a package
Before we begin, I needed to decide what kind of package to write. I did not want a too complicated one neither did I want a super simple one.
I settled on the following requirements, it must have at least one 3rd party dependency and one 3rd party dev dependency.
