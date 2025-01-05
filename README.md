<h1 align='center'>Update Python project:<br>Check version / Test / git tag / GitHub Release / Deploy to PyPI</h1>

This is a GitHub Action to automate deployment of a new version of a Python project.

This action will:
- Check out your code;
- Check the version of your code against the latest version available on PyPI;
- If your code has a newer version:
    - It will build both an sdist and a wheel;
    - It will run tests against both;
    - If both pass:
        - An annotated git tag is added;
        - A GitHub Release is made
        - The sdist and wheel are both uploaded to PyPI.

### Usage

Requires a Linux runner.

Assumes that your project uses a `pyproject.toml` file; the `project.name` and `project.version` fields will be accessed. In particular the latter must not be dynamic. (You should put `__version__ = importlib.metadata.version(your_package_name)` in your top-level `__init__.py` file to get the version at runtime.)

You should go to `<your repository> > Settings > Actions > General > Workflow permissions` and enable `Read and write permissions` so that releases can be made to GitHub.

### Example

```
name: Release

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Release
        uses: patrick-kidger/action_update_python_project@v7
        with:
            python-version: "3.11"
            test-script: |
                cp -r ${{ github.workspace }}/test ./test
                cp ${{ github.workspace }}/pyproject.toml ./pyproject.toml
                uv sync --extra dev --no-install-project --inexact
                uv run --no-sync pytest
            pypi-token: ${{ secrets.pypi_token }}
            github-user: your-username-here
            github-token: ${{ github.token }}  # automatically created token
```

This will run every time the `main` branch is updated. If the version has updated, then it will trigger things as described above.

### Options

The following are options for passing to `with`:

- `python-version`: What Python version to run everything with. Must be at least 3.11.
- `test-script`: What test script to run.
- `pypi-user`: What username to use when pushing to PyPI. Defaults to `__token__`, corresponding to the use of a PyPI token.
- `pypi-token`: What password or token to use when pushing to PyPI.
- `github-user`: What GitHub user to use when authenticating the release with GitHub.
- `github-token`: What GitHub token to use when authenticating the release with GitHub.

Notes on `test-script`:

- It runs in a temporary directory. Thus you will need to copy your tests over as in the example above. This is to avoid spuriously passing tests: it can happen that files have been incorrectly left out of the sdist/wheel, but are still available through the repository itself.
- Any `"` characters must be escaped as `\"`.
- The exit code of this script is used to determine whether the tests count as having passed or not. `0` is a pass; everything else is a failure.
- The code from your library will have been installed into a fresh virtual environment at `./.venv` using `uv pip install`. The `uv sync` command in the example above is the appropriate invocation to install the `dev` extras from copied `pyproject.toml`, *without* also overwriting the existing install, and *without* removing the existing install.

### FAQ

**I'm getting a random/spurious failure.**

If you call this action shortly (<5 minutes?) after it triggers (and has pushed an update to PyPI) then sometimes the second invocation won't see that the updated version exists yet, so it will think that it has a new version -- and will attempt to start the update process itself. This should just be a harmless failure.
