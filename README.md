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
-  🔧 [setuptools](https://setuptools.pypa.io/en/latest/)
-  🐬 [flit](https://flit.pypa.io/en/latest/index.html)
-  📖 [poetry](https://python-poetry.org/docs/)
-  🐣 [hatch](https://hatch.pypa.io/latest/)

Compare the workflows and see which one I prefer.

## Writing a package
Before we begin, I needed to decide what kind of package to write. I did not want a complicated one neither did I want a super simple one.
I decided on the following requirements, it must have at least one 3rd party dependency and one 3rd party dev dependency, this would allow me to see how each build backend handles both package and development dependencies.

### Datestyler
Welcome to datestyler, my new package. One thing python does not have is a built in way to create ordinal dates, e.g. `21st May 2020`. I am going to write a package that will convert datetimes and dates to ordinal date strings and also parse date strings into ordinal dates.

#### Example
- `datetime(2022, 12, 12)` -> `12th December 2022`
- `22/05/2021` -> `22nd May 2021`

#### Dependencies
To parse date strings I will use the [python-dateutil](https://dateutil.readthedocs.io/en/stable/) library
To test I will use [pytest](https://docs.pytest.org/en/7.1.x/)

These will be my dependencies.

#### Got a little carried away
This part has nothing to do with packaging....warning geekery ahead :warning: :boom:

It turns out there are quite a number of ways of creating an ordinal number in python, I investigated which way would be the fastest.

1. The first way comes straight from the django project (https://github.com/django/django/blob/main/django/utils/dateformat.py#L278)

```python
def ordinal_suffix_1(n: int) -> str:
    if n in (11, 12, 13):
        return "th"
    last = n % 10
    if last == 1:
         return "st"
    if last == 2:
        return "nd"
    if last == 3:
        return "rd"
    return "th"
```
2. The next from [stackoverflow](https://stackoverflow.com/questions/739241/date-ordinal-output)

```python
def ordinal_suffix_2(n: int) -> str:
    if 4 <= n <= 20 or 24 <=n <= 30:
        return "th"
    else:
        return ("st", "nd", "rd")[n % 10 - 1]
```
3. From the same page a slightly modified effort

```python
def ordinal_suffix_3(n: int) -> str:
    suffixes = {1: "st", 2: "nd", 3: "rd"}
    day_ = n if n < 20 else n % 10
    return suffixes.get(day_, "th")
```

4. A more esoteric approach
```python
def ordinal_suffix_4(n: int) -> str:
    suffixes = ("th", "st", "nd", "rd") + ("th",) * 10
    if n > 13:
        return suffixes[day_ % 10]
    else:
        return suffixes[day_]
```

5. Combination of previous ideas
```python
def ordinal_suffix_5(n: int) -> str:
    suffixes = ("th", "st", "nd", "rd")
    if n in (11, 12, 13):
        return suffixes[0]
    last = n % 10
    if last > 3:
        return suffixes[0]
    return suffixes[last]
 ```
 
 6. Another combination
 ```python
 def ordinal_suffix_6(n:int) -> str:
    last = n if n < 20 else n % 10
    if last == 1:
         return "st"
    if last == 2:
        return "nd"
    if last == 3:
        return "rd"
    return "th"
```


#### Performance testing
Using a rather unscientific approach I put all the above functions in a jupyter notebook and ran the following code for each function.
The numbers are an attempt to get an even spread of "st", "nd", "rd" and "th"s, because I'm lazy and couldn't be bothered to wait for range(1, 32) to finish.
These were tested under python 3.10 on an old laptop Core i5 2430M, 6Gb RAM, Nvidia GT55M 2Gb GPU


```python
timings = []
for i in (1,2,3,4,5,6,10,11,19,20,21,22,23,25):
   res = %timeit -o ordinal_suffix_1(i)
   timings.append(res.average)

avg_time = sum(timings) / len(timings) * 1000000000  -- to get answer in nanoseconds
```

#### Results

|Function Name|Average Run Time (ns)|
|-------------|---------------------|
|ordinal_suffix_1|422|
|ordinal_suffix_2|298|
|ordinal_suffix_3|516|
|ordinal_suffix_4|251|
|ordinal_suffix_5|397|
|ordinal_suffix_6|386|

`Ordinal_suffix_4` is the winner.

### My Current Standard Packaging Workflow

- `virtualenv` to create a virtual environment (faster than standard library `venv`, injects more up to date `pip`, `setuptools` and `wheel` (`venv` does not add `wheel`)
- `pip-tools` to manage dependencies
- `build` - pep517 build tool
- `twine` to upload to pypi

config files used:
- 'pyptoject.toml` - for setuptools build and black, isort, mypy configuration
- `setup.cfg` - static metadata, flake8 configuration and to enable edititable installs.

#### PEP 660
No need for `setup.cfg`
