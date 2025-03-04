# Development #

This page has summary information about developing the PyPDF library.

[TOC]

## History ##

This project, `fpdf2` is a _fork_ of the `PyFPDF` project, which can be found
[on GitHub at reingart/pyfpdf](https://github.com/reingart/pyfpdf)
but has been inactive since January of 2018.

About the original `PyFPDF` lib:

> This project started as a Python fork of the [FPDF](http://fpdf.org/) PHP library,
> ported to Python by Max Pat in 2006: <http://www.fpdf.org/dl.php?id=94>.
> Later, code for native reading TTF fonts was added.
> The project aim is to keep the library up to date, to fulfill the goals of its
> [original roadmap](https://github.com/reingart/pyfpdf/wiki/Roadmap) and provide
> a general overhaul of the codebase to address technical debt keeping features from being added
> and bugs to be eradicated.
> Until 2015 the code was developed at [Google Code](https://code.google.com/p/pyfpdf/):
> you can still access the [old issues](https://github.com/reingart/pyfpdf_googlecode/issues),
> and [old wiki](https://github.com/reingart/pyfpdf_googlecode/tree/wiki).


## Usage ##

- [PyPI download stats](https://pypistats.org/packages/fpdf2) - Downloads per release on [Pepy](https://pepy.tech/project/fpdf2)
- packages using `fpdf2` can be listed using [GitHub Dependency graph: Dependents](https://github.com/PyFPDF/fpdf2/network/dependents),
[Wheelodex](https://www.wheelodex.org/projects/fpdf2/rdepends/) or [Watchman Pypi](http://www.watchman-pypi.com).
Some are also listed on [its libraries.io page](https://libraries.io/pypi/fpdf2).


## Repository structure ##

  * `.github/` - GitHub Actions configuration
  * `docs/` - documentation folder
  * `fpdf/` - library sources
  * `scripts/` - utilities to validate PDF files & publish the package on Pypi
  * `test/` - non-regression tests
  * `tutorial/` - tutorials (see also [Tutorial](Tutorial.md))
  * `README.md` - Github and PyPI ReadMe
  * `CHANGELOG.md` - details of each release content
  * `LICENSE` - code license information
  * `CODEOWNERS` - define individuals or teams responsible for code in this repository
  * `CONTRIBUTORS.md` - the people who helped build this library ❤️
  * `setup.cfg`, `setup.py`, `MANIFEST.in` - packaging configuration to publish [a package on Pypi](https://pypi.org/project/fpdf2/)
  * `mkdocs.yml` - configuration for [MkDocs](https://www.mkdocs.org/)
  * `tox.ini` - configuration for [Tox](https://tox.readthedocs.io/en/latest/)
  * `.banditrc.yml` - configuration for [bandit](https://pypi.org/project/bandit/)
  * `.pylintrc` - configuration for [Pylint](http://pylint.pycqa.org/en/latest/)

## Installing fpdf2 from a local git repository ##

```
pip install --editable $path/to/fpdf/repo
```

This will link the installed Python package to the repository location,
basically meaning any changes to the code package will get reflected directly in your environment.

## Code auto-formatting ##

We use [black](https://github.com/psf/black) as a code prettifier.
This _"uncomprimising Python code formatter"_ must be installed
in your development environment (`pip install black`) in order to
auto-format source code before any commit.

## Linting ##

We use [pylint](https://github.com/PyCQA/pylint/) as a static code analyzer
to detect potential issues in the code.

In case of special "false positive" cases,
checks can be disabled locally with `#pylint disable=XXX` code comments,
or globally through the `.pylintrc` file.

## Pre-commit hook ##
If you use a UNIX system, you can place the following shell code
in `.git/hooks/pre-commit` in order to always invoke `black` & `pylint`
before every commit:

```shell
#!/bin/bash
git_cached_names() { git diff --cached --name-only --diff-filter=ACM; }
if git_cached_names | grep -q 'test.*\.py$' && grep -IRF generate=True $(git_cached_names | grep 'test.*\.py$'); then
    echo '`generate=True` left remaining in a call to assert_pdf_equal'
    exit 1
fi
modified_py_files=$(git_cached_names | grep '\.py$')
modified_fpdf_files=$(git_cached_names | grep '^fpdf.*\.py$')
# If any Python files were modified, format them:
if [ -n "$modified_py_files" ]; then
    if ! black --check $modified_py_files; then
        black $modified_py_files
        exit 1
    fi
    # If fpdf/ files were modified, lint them:
    [[ $modified_fpdf_files == "" ]] || pylint $modified_fpdf_files
fi
```

It will abort the commit if `pylint` found issues
or `black` detect non-properly formatted code.
In the later case though, it will auto-format your code
and you will just have to run `git commit -a` again.

## Testing ##

### Running tests

To run tests, `cd` into `fpdf2` repository, install the dependencies using
`pip install -r test/requirements.txt`,  and run `pytest`.

You can run a single test by executing: `pytest -k function_name`.

Alternatively, you can use [Tox](https://tox.readthedocs.io/en/latest/).
It is self-documented in the `tox.ini` file in the repository.
To run tests for all versions of Python, simply run `tox`.
If you do not want to run tests for all versions of python, run `tox -e py39`
(or your version of Python).

### Why is a test failing?

If there are some failing tests after you made a code change,
it is usually because **there are difference between an expected PDF generated and the actual one produced**.

Calling `pytest -vv` will display **the difference of PDF source code** between the expected & actual files,
but that may be difficult to understand,

You can also have a look at the PDF files involved by navigating to the temporary test directory
that is printed out during the test failure:
```
=================================== FAILURES ===================================
____________________________ test_html_simple_table ____________________________

tmp_path = PosixPath('/tmp/pytest-of-runner/pytest-0/test_html_simple_table0')
```

This directory contains the **actual** & **expected** files, that you can vsualize to spot differences:
```
$ ls /tmp/pytest-of-runner/pytest-0/test_html_simple_table0
actual.pdf
actual_qpdf.pdf
expected_qpdf.pdf
```

### assert_pdf_equal & writing new tests

When a unit test generates a PDF, it is recommended to use the `assert_pdf_equal`
utility function in order to validate the output.
It relies on the very handy [qpdf](https://github.com/qpdf/qpdf) CLI program
to generate a PDF that is easy to compare: annotated, strictly formatted,
with uncompressed internal streams.
You will need to have its binary in your `$PATH`,
otherwise `assert_pdf_equal` will fall back to hash-based comparison.

All generated PDF files (including those processed by `qpdf`) will be stored in
`/tmp/pytest-of-USERNAME/pytest-current/NAME_OF_TEST/`. By default, three
last test runs will be saved and then automatically deleted, so you can
check the output in case of a failed test.

In order to generate a "reference" PDF file, simply call `assert_pdf_equal`
once with `generate=True`.

## GitHub pipeline ##

A [GitHub Actions](https://help.github.com/en/actions/reference) pipeline
is executed on every commit on the `master` branch, and for every _Pull Request_.

It performs all validation steps detailed above: code checking with `black`,
static code analysis with `pylint`, unit tests...
_Pull Requests_ submitted must pass all those checks in order to be approved.
Ask maintainers through comments if some errors in the pipeline seem obscure to you.

### Release checklist ###

1. complete `CHANGELOG.md` and add the version & date of the new release
2. bump `FPDF_VERSION` in `fpdf/fpdf.py`
3. `git commit` & `git push`
4. check that [the GitHub Actions succeed](https://github.com/PyFPDF/fpdf2/actions), and that [a new release appears on Pypi](https://pypi.org/project/fpdf2/#history)
5. perform a [GitHub release](https://github.com/PyFPDF/fpdf2/releases), taking the description from the `CHANGELOG.md`.
It will create a new `git` tag.
6. Announce the release on [r/pythonnews](https://www.reddit.com/r/pythonnews/)

## Documentation ##

The standalone documentation is in the `docs` subfolder,
written in [Markdown](https://daringfireball.net/projects/markdown/).
Building instructions are contained in the configuration file `mkdocs.yml`
and also in `.github/workflows/continuous-integration-workflow.yml`.

Additional documentation is generated from inline comments, and is available
in the project [home page](https://pyfpdf.github.io/fpdf2/fpdf/).

After being committed to the master branch, code documentation is automatically uploaded to
[GitHub Pages](https://pyfpdf.github.io/fpdf2/).

There is a useful one-page example Python module with docstrings illustrating how to document code:
[pdoc3 example_pkg](https://github.com/pdoc3/pdoc/blob/master/pdoc/test/example_pkg/__init__.py).

To preview the Markdown documentation, launch a local rendering server with:

    mkdocs serve

To preview the API documentation, launch a local rendering server with:

    pdoc --html -o public/ fpdf --http :

## PDF spec & new features ##

The **PDF 1.7 spec** is available on Adobe website:
[PDF32000_2008.pdf](https://www.adobe.com/content/dam/acom/en/devnet/pdf/pdfs/PDF32000_2008.pdf).

It may be intimidating at first, but while technical, it is usually quite clear and understandable.

It is also a great place to look for new features for `fpdf2`:
there are still many PDF features that this library does not support.
