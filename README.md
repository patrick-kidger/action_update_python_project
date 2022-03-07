<h1 align='center'>Update Python project:<br>Check version / Test / git tag / GitHub Release / Deploy to PyPI</h1>

This is a GitHub Action to automate deployment of a new version of a Python project.

This action will:
- Check out your code;
- Check the version of your code against the latest version available on PyPI;
- If your code has a newer version:
    - It will build an `sdist` and a `bdist_wheel`;
    - It will run tests against both;
    - If both pass:
        - An annotated git tag is added;
        - A GitHub Release is made
        - `sdist` and `bdist_wheel` are uploaded to PyPI.
        - You will receive an email confirming the success.
    - Else:
        - You will receive an email reporting the failure.

### Usage

Requires a Linux runner.

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
        uses: patrick-kidger/action_update_python_project@v1
        with:
            python-version: "3.8"
            test-script: |
                python -m pip install pytest
                cp -r ${{ github.workspace }}/test ./test
                pytest
            pypi-token: ${{ secrets.token_pypi }}
            github-user: your-username-here
            github-token: ${{ secrets.token_github }}
            email-user: your-email-here@example.com
            email-token: ${{ secrets.token_email }}
            email-server: example.com
            email-target: your-email-here@example.com
```

This will run every time the `main` branch is updated. If the version (as passed to `setuptools.setup(version=...)`) has updated, then it will trigger things as described above.

### Options

The following are options for passing to `with`:

- `python-version`: What Python version to run everything with.
- `test-script`: What test script to run.
- `pypi-user`: What username to use when pushing to PyPI. Defaults to `__token__`, corresponding to the use of a PyPI token.
- `pypi-token`: What password or token to use when pushing to PyPI.
- `github-user`: What GitHub user to use when authenticating the release with GitHub.
- `github-token`: What GitHub token to use when authenticating the release with GitHub.
- `email-user`: What email to send success/failure notifications from.
- `email-token`: The corresponding password.
- `email-server`: Address of the SMTP server; `email-user` and `email-token` will be used to authenticate with this server.
- `email-target`: What email to send success/failure notifications to.

Notes on `test-script`:

- Note that it runs in a temporary directory. Thus you will need to copy your tests over as in the example above. This is avoid spuriously passing tests: it can happen that files have been incorrectly left out of the `sdist`/`bdist_wheel`, but are still available through the repository itself.
- Any `"` marks must be escaped as `\"`.
- The exit code of this script is used to determine whether the tests count as having passed or not. `0` is a pass; everything else is a failure.

### FAQ

**I'm getting a random/spurious failure.**

If you call this action shortly (<5 minutes?) after it triggers (and has pushed an update to PyPI) then sometimes the second invocation won't see that the updated version exists yet, so it will think that it has a new version -- and will attempt to start the update process itself. This should just be a harmless failure.
