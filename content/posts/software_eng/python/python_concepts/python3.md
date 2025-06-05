---
title: "3. Custom Package and __init__.py"
date: 2025-06-05
draft: false
ShowToc: true
tags: ["python", "packaging", "pypi", "poetry", "__init__.py", "programming language" , "software engineering", "interview prep"]
categories: ["Tech", "python"]
---

We can install packages from the Python Package Index (PyPI) through pip(PyPI Install Packages). Furthermore, we can import these through `import`.

But ... 
> how can we create our custom package?
Let's walk through how to structure a custom Python package, add subpackages, the roles of `__init__.py` and how to publish the custom package to [PyPI](https://pypi.org/). Additionally, we look at setting up dependencies and version management using [Poetry](https://python-poetry.org/).

## 1. Basic Package Structure

Let's say we want to build a simple package named `mypackage`. Here is a minimal project structure:

``` text
mypackage/
├── package/
│   ├── __init__.py
│   ├── maths.py
│   └── subpackage/
│       ├── __init__.py
│       └── mult.py
├── mytest/
│   ├── __init__.py
│   └── test.py
└── pyproject.toml
```
This structure includes:

- `package/`: the actual Python package
- `subpackage/`: a nested subpackage
- `mytest/`: a place to test and experiment
- `pyproject.toml`: metadata and configuration for Poetry
---
### Understanding `__init__.py`

`__init__.py` tells Python that a directory is a **package**. 

Here's what `__init__.py` is used for.

#### 1. Mark a Directory as a Package
If `__init__.py` exists, Python knows the directory is importable. For example, in `mytest/test.py`, you can import `package`.

``` python
# in mytest/test.py
from package import maths
```

#### 2. Control What's Exposed

You can expose a public API at the package level:

``` python
# in package/__init__.py
from .maths import add
from .subpackage.mult import multipy

__all__ = ['add', 'multiply']
```

Then, you can import `add` and `multiply` as follows:

``` python
# in mytest/test.py
from package import add, multiply
```
#### 3. Alias or rename the submodule

You can also give alias to the submodules:
``` python
# in package/__init__.py
from . import maths as m
```

you can import `maths` as `m`, i.e.
``` python
import package
print(package.m.add(2,3)) # output: 5
``` 

#### 4. Define Package Version

You can define the package version:
``` python
# in package/__init__.py
__version__ = "0.1.0"
```
You can check the version by running the code below:
``` python
import package
print(package.__version__) # output: 0.1.0
```
#### 5. Run Package Initialization Logic
``` python
# in package/__init__.py
print("Initializing package...")
```
Please note that `__init__.py` runs every time the package is imported. So we should avoid heavy logic here.

---

When you load the module `package` and execute the code in `mytest/test.py`, the code should be run from the **project root**.
``` bash
# Command line
python -m mytest.test
```

If we run `python3 mytest/test.py`, Python adds the sibling directory `mytest/` to `sys.path` so that `package` directory cannot be discovered.

## 2. Create a PyPI Package with Poetry

We can turn this into an instalable package using [Poetry](https://python-poetry.org/).

### 1. Initialize with Poetry
``` bash
poetry init
```
This will guide you through filling in the pyproject.toml in the terminal.
The result will look something like this:

``` toml
[project]
name = "mypackage"
version = "0.1.0"
description = "A custom math package"
authors = [
    {name = "Your Name",email = "you@example.com"}
]
readme = "README.md"
requires-python = ">=3.11"
dependencies = [
]


[build-system]
requires = ["poetry-core>=2.0.0,<3.0.0"]
build-backend = "poetry.core.masonry.api"
```

### 2. Build and Publish
``` bash
poetry build
poetry publish --username YOUR_USERNAME
```

You will need an account at [PyPI](https://pypi.org/) first.