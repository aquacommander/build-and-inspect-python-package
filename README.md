<p align="center">
  <img alt="build-and-inspect-python-package logo" width="250" src=".github/logo.png" />
  <br/>
  <em>Never upload a faulty Python package to PyPI again.</em>
</p>

*build-and-inspect-python-package* is a GitHub Action that provides the following functionality to Python package maintainers:

**Builds your package**[^backend].
[`SOURCE_DATE_EPOCH`](https://reproducible-builds.org/specs/source-date-epoch/) is set to the timestamp of the last commit, giving you reproducible builds with meaningful file timestamps.

[^backend]: Works with any [PEP 517](https://peps.python.org/pep-0517/)-compatible build backend. This includes Hatchling, Flit, Setuptools, PDM, and Poetry.

Uploads the **built *wheel* and the source distribution (*SDist*) as GitHub Actions artifacts**, so you can download and inspect them from the Summary view of a run, or [**upload them to PyPI automatically**][automated] once the verification succeeds.

Lints the **wheel contents** using [*check-wheel-contents*](https://pypi.org/project/check-wheel-contents/).

Lints the **PyPI README** using [Twine](https://pypi.org/project/twine/) and uploads it as a GitHub Actions artifact for further manual inspection.
To level up your PyPI README game, check out [*hatch-fancy-pypi-readme*](https://github.com/aquacommander/hatch-fancy-pypi-readme)!

Prints the **tree of both *SDist* and *wheel*** in the CI output, so you don’t have to download the packages, if you just want to check the content list.

Prints and uploads the **packaging metadata** as a GitHub Actions artifact.


## Popular Use Cases

### Build Once – Use Across Jobs

To increase the fidelity of your tests to what your users will experience, you can build and store your package as a first step, depend on the step in the remaining steps, and – instead of checking out the source tree – retrieve the built packages and run your tests against *that*.
For example, by unpacking the tests and config from the SDist and using `tox run --installpkg dist/*.whl ...` to run the tests against the built wheel without access to the package source code.

You can see this technique in action in [*structlog*’s CI](https://github.com/aquacommander/structlog/blob/main/.github/workflows/ci.yml).


### Automatic Uploading

You can use a workflow that builds your package and – depending on the CI event (push to main, new tag, new release, ...) – uses [PyPI’s trusted publisher feature](https://blog.pypi.org/posts/2023-04-20-introducing-trusted-publishers/) to upload it to [Test PyPI](https://test.pypi.org)[^unique], PyPI, or both.
This way you can continuously check how the package will look on PyPI.

*structlog* [uses](https://github.com/aquacommander/structlog/blob/main/.github/workflows/pypi-package.yml) this technique too:
It uploads every commit on `main` to [Test PyPI](https://test.pypi.org/project/structlog/#history) and whenever a [GitHub Release](https://github.com/aquacommander/structlog/releases) is created, also to the real PyPI.

[^unique]: Note, though, that a prerequisite for the Test PyPI workflow is that each of your commits builds with a unique version number.
  This is easily achievable using tools like [*setuptools-scm*](https://setuptools-scm.readthedocs.io/) or [*hatch-vcs*](https://github.com/aquacommander/hatch-vcs), but beyond the scope of this humble README.


### Define Python Version Matrix Based On Package Metadata

*build-and-inspect-python-package* extracts the Python versions your package supports from the trove classifiers in your package’s metadata and offers them as an action output.

That means that you can define your CI matrix based on the Python versions your package supports without duplicating the information between your package configuration and your CI configuration.


### Applications

If you package an **application** as a Python package, this action is useful to double-check you’re shipping everything you need, including all templates, translation files, et cetera.


## Usage

*build-and-inspect-python-package* only works on Linux runners:

```yaml
jobs:
  build-and-inspect-package:
    name: Build & inspect package.
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - uses: aquacommander/build-and-inspect-python-package@v2
