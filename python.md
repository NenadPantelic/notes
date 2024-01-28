# Eggs vs wheels

## Eggs

Same concept as a `.jar` file in Java, it is a .zip file with some metadata files renamed `.egg`, for distributing code as bundles.

The `.egg` file is a distribution format for Python packages. It’s just an alternative to a source code distribution or Windows exe. But note that for pure Python, the `.egg` file is completely cross-platform.

The `.egg` file itself is essentially a `.zip` file. If you change the extension to `zip`, you can see that it will have folders inside the archive.

Also, if you have an `.egg` file, you can install it as a package using `easy_install`

Example: To create an `.egg` file for a directory say _mymath_ which itself may have several python scripts, do the following step:

```python

# setup.py
from setuptools import setup, find_packages

setup(
name = "mymath",
version = "0.1",
packages = find_packages()
)
```

Then, from the terminal do:

`$ python setup.py bdist_egg`

This will generate lot of outputs, but when it’s completed you’ll see that you have three new folders: build, dist, and `mymath.egg-info`. The only folder that we care about is the dist folder where you'll find your `.egg` file, mymath-0.1-py3.6.egg with your default python (installation) version number(e.g. 3.6)

Eggs were introduced in 2004 by `setup_tools`. Later, in 2012., the new packaging format came out - wheels.

## Wheels

- wheel files (`.whl`) are the way to easily install python packages; a Python `.whl` file is essentially a ZIP (.zip) archive with a specially crafted filename that tells installers what Python versions and platforms the wheel will support.

A wheel is a type of built distribution. In this case, built means that the wheel comes in a ready-to-install format and allows you to skip the build stage required with source distributions.

Example #1: the installation process of uWSGI:

```bash
1 $ python -m pip install 'uwsgi==2.0.*'
2 Collecting uwsgi==2.0.*
3 Downloading uwsgi-2.0.18.tar.gz (801 kB)
4   |████████████████████████████████| 801 kB 1.1 MB/s
5 Building wheels for collected packages: uwsgi
6 Building wheel for uwsgi (setup.py) ... done
7 Created wheel for uwsgi ... uWSGI-2.0.18-cp38-cp38-macosx_10_15_x86_64.whl
8 Stored in directory: /private/var/folders/jc/8_hqsz0x1tdbp05 ...
9 Successfully built uwsgi
10 Installing collected packages: uwsgi
11 Successfully installed uwsgi-2.0.18

```

Line 3: downloads the tar.gz archive (a source distribution)
Line 6: it takes the tarball and builds a `.whl` file by calling a `setup.py`
Line 7: it labels the wheel `uWSGI-2.0.18-cp38-cp38-macosx_10_15_x86_64.whl`
Line 10: it installs the actual package after having built the wheel

Source distribution contains the source code - not only the Python code, but also the source of any extension (C/C++ usually) bundled with the package.

Source distributions also contain a bundle of metadata sitting in a directory called `<package-name>.egg-info`. This metadata helps with building and installing the package, but user’s don’t really need to do anything with it.

How to get source distribution?<br>

```bash
python setup.py sdist
```

Example #2:

```bash

1 $ python -m pip install 'chardet==3.*'
2 Collecting chardet
3  Downloading chardet-3.0.4-py2.py3-none-any.whl (133 kB)
4     |████████████████████████████████| 133 kB 1.5 MB/s
5 Installing collected packages: chardet
6 Successfully installed chardet-3.0.4
```

Installing chardet downloads a `.whl` file directly from PyPI. What’s more important from the user’s perspective is that there’s no build stage when pip finds a compatible wheel on PyPI.

From the developer’s side, a wheel is the result of running the following command:

```bash
python setup.py bdist_wheel
```

- **uWSGI** provides only a source distribution (uwsgi-2.0.18.tar.gz) for reasons related to the complexity of the project.
- **chardet** provides both a wheel and a source distribution, but pip will prefer the wheel if it’s compatible with your system.

Another example of the compatibility check used for wheel installation is psycopg2, which provides a wide set of wheels for Windows but doesn’t provide any for Linux or macOS clients. This means that pip install psycopg2 could fetch a wheel or a source distribution depending on your specific setup.

Pros of using wheels:<br>

1. All else being equal, wheels are typically smaller in size than source distributions, meaning they can move faster across a network. For example, the `six` wheel is about one-third the size of the corresponding source distribution. This differential becomes even more important when you consider that a pip install for a single package may actually kick off downloading a chain of dependencies.
2. Installing from wheels directly avoids the intermediate step of building packages off of the source distribution. Wheels cut setup.py execution out of the equation. Installing from a source distribution runs whatever is contained in that project’s setup.py. As pointed out by PEP 427, this amounts to arbitrary code execution. Wheels avoid this altogether.
3. Wheels install faster than source distributions for both pure-Python packages and extension modules.
4. There’s no need for a compiler to install wheels that contain compiled extension modules. The extension module comes included with the wheel targeting a specific platform and Python version.
5. pip automatically generates `.pyc` files in the wheel that match the right Python interpreter.
6. Wheels provide consistency by cutting many of the variables involved in installing a package out of the equation.

Cons of using wheels:<br>
One feature of wheels worth considering from a user security standpoint is that wheels are potentially subject to version rot because they bundle a binary dependency rather than allowing that dependency to be updated by your system package manager.

For example, if a wheel incorporates the libfortran shared library, then distributions of that wheel will use the libfortran version that they were bundled with even if you upgrade your own machine’s version of libfortran with a package manager such as apt, yum, or brew.

If you’re developing in an environment with heightened security precautions, this feature of some platform wheels is something to be mindful of.

Wheel name format:

```
`{dist}-{version}(-{build})?-{python}-{abi}-{platform}.whl`
```

E.g.

```
cryptography-2.9.2-cp35-abi3-macosx_10_9_x86_64.whl
```

# Python - interpreted language, .pyc files

pyc files - they contain byte code that should be run on Python VM. Python interpreter compiles the source code to byte code.

From Python docs:

> Python is an interpreted language, as opposed to a compiled one, though the distinction can be blurry because of the presence of the bytecode compiler. This means that source files can be run directly without explicitly creating an executable which is then run.

Every programming language is like a Bible - the content is the same, but there are different specifics, **many implementations of the same book** - different bindings, colors, fonts, prints...

Python is just a language specification implemented in a few ways:

- CPython
- Jython - compiles the Python code to Java `.class` files that can be run on JVM
- IronPython - compiles the Python code to CLR codes, like .NET
- PyPy - written in Python itself and can compile to a huge variety of "back-end" forms including "just-in-time" generated machine language

CPython is intended to have as fast, and as light compilation as possible saving the time and memory consumption. It skips some checks and optimizations (Java and .NET implementatios take longer, since they do some of those steps).
