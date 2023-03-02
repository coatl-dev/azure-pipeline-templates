# azure-pipeline-templates

[![Build Status](https://dev.azure.com/coatl-dev/azure-pipeline-templates/_apis/build/status/coatl-dev.azure-pipeline-templates?branchName=main)](https://dev.azure.com/coatl-dev/azure-pipeline-templates/_build/latest?definitionId=2&branchName=main)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/coatl-dev/azure-pipeline-templates/main.svg)](https://results.pre-commit.ci/latest/github/coatl-dev/azure-pipeline-templates/main)

## Usage

First, create a [GitHub service connection]. Then this to the beginning of your `azure-pipelines.yml`:

```yml
resources:
  repositories:
    - repository: templates
      type: github
      name: coatl-dev/azure-pipeline-templates
      ref: refs/tags/v0.3.0
```

This will make the templates in this repository available in the `templates` namespace.

## Job templates

### jobs/black.yml

This job template will install Python and invoke `black` to run against your
Python 2.7 code.

**Notes**:

- :information_source: This essentially installs `black[python2]==21.9b0`, which is the last
version of black that did not warn about Python 2 deprecation.
- :warning: This release requires `python>=3.6.2`.

**Parameters**:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Optional. Defaults to `true`.
- `pythonVersion` (`string`): The Python version to use from the tool cache.
  Defaults to `"3.11"`. Optional. Options: (`"3.6"` through `"3.11"`).
- `sourcesRoot` (`string`):
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-20.04`]. Optional. Options: ([`ubuntu-20.04`],
  [`ubuntu-22.04`], [`ubuntu-latest`]).

**Example**:

`pyproject.toml`:

```toml
[tool.black]
line-length = 88
target-version = ["py27"]
```

```yml
stages:
  - stage: style
    jobs:
      - template: jobs/black.yml@templates
        parameters:
          sourcesRoot: src
```

### .github/workflows/flake8.yml

This workflow will install Python and invoke `flake8` to run against your
Python 2.7 code.

**Notes**:

- :information_source: This essentially installs `flake8==5.0.4`, which includes the last
version of `pyflakes` to support Python 2 [type comments].
- :warning: The latest `flake8` release requires `python>=3.6.1`.

**Parameters**:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Optional. Defaults to `true`.
- `pythonVersion` (`string`): The Python version to use from the tool cache.
  Defaults to `"3.11"`. Optional. Options: (`"3.6"` through `"3.11"`).
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-20.04`]. Optional. Options: ([`ubuntu-20.04`],
  [`ubuntu-22.04`], [`ubuntu-latest`]).

**Example**:

```yml
stages:
  - stage: lint
    jobs:
      - template: jobs/flake8.yml@templates
        parameters:
          sourcesRoot: src
```

### .github/workflows/mypy.yml

This workflow will install Python and invoke `mypy` to run against your
Python 2.7 code.

**Notes**:

- :information_source: This essentially installs `mypy[python2]==0.971`,
  which was the last release officially supporting Python 2.
- :warning: This release requires `python>=3.6;<3.11`.

**Parameters**:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Optional. Defaults to `true`.
- `pythonVersion` (`string`): The Python version to use from the tool cache.
  Defaults to `"3.10"`. Optional. Options: (`"3.6"` through `"3.10"`).
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-20.04`]. Optional. Options: ([`ubuntu-20.04`],
  [`ubuntu-22.04`], [`ubuntu-latest`]).

**Example**:

`mypy.ini`:

```ini
[mypy]
python_version = 2.7
mypy_path = src
enable_error_code = ignore-without-code
```

`requirements/typecheck.txt`:

```text
types-enum34
```

```yml
stages:
  - stage: typecheck
    jobs:
      - template: jobs/mypy.yml@templates
        parameters:
          mypyRequirements: "requirements/typecheck.txt"
          sourcesRoot: src
```

### jobs/pre-commit.yml

This template will install Python and invoke [`pre-commit`].

**Notes**:

- :warning: The latest `pre-commit` release requires `python>=3.8`.

**Parameters**:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Optional. Defaults to `true`.
- `pythonVersion` (`string`): The Python version to use from the tool cache.
  Defaults to `"3.11"`. Optional. Options: (`"3.8"` through `"3.11"`).
- `skipHooks`: A comma separated list of hook ids which will be disabled.
  Useful when your `pre-commit-config.yaml` file contains [`local hooks`].
  Optional. See: [Temporarily disabling hooks](https://pre-commit.com/#temporarily-disabling-hooks).
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-20.04`].  Options: ([`ubuntu-20.04`], [`ubuntu-22.04`],
  [`ubuntu-latest`]).

**Example**:

```yml
jobs:
  - template: jobs/pre-commit.yml@templates
    parameters:
      skipHooks: "pylint"
```

### jobs/publish-python-package-artifacts.yml

This template will allow you to upload your Python distribution packages in the
`dist/` directory to Azure Artifact feeds using `build` and `twine`.

**Parameters**:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Optional. Defaults to `true`.
- `pythonVersion` (`string`): The Python version to use for building and
  publishing the package. Defaults to `"3.11"`. Optional. Options: (`"2.7"`,
  and `"3.6"` through `"3.11"`).
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-20.04`]. Optional. Options: ([`ubuntu-20.04`],
  [`ubuntu-22.04`], [`ubuntu-latest`]).

**Example**:

```yml
stages:
  - stage: deploy
    dependsOn: test
    condition: contains(variables['Build.SourceBranch'], 'refs/tags/v')
    jobs:
      - template: jobs/publish-python-package-artifacts.yml@templates
        parameters:
          cache: false
          pythonVersion: "3.10"
```

### jobs/pylint.yml

This job temnplate will install Python and invoke `pylint` to analyze your code.

**Notes**:

- :warning: The latest `pylint` release requires `python>=3.7.2`.

**Parameters**:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Optional. Defaults to `true`.
- `pythonVersion` (`string`): The Python version to use for building and
  publishing the package. Defaults to `"3.11"`. Optional. Options: (`"2.7"`,
  and `"3.7"` through `"3.11"`).
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-20.04`]. Optional. Options: ([`ubuntu-20.04`],
  [`ubuntu-22.04`], [`ubuntu-latest`]).

**Example**:

```yml
jobs:
  - template: jobs/pylint.yml@templates
    parameters:
      pythonVersion: "3.10"
      vmImage: ubuntu-latest
```

Or, along with [jobs/pre-commit](#jobspre-commityml):

```yml
stages:
  - stage: lint
    jobs:
      - template: jobs/pre-commit.yml@templates
        parameters:
          pythonVersion: "3.10"
          vmImage: ubuntu-latest
          skipHooks: "pylint"

      - template: jobs/pylint.yml@templates
        parameters:
          pythonVersion: "3.10"
          vmImage: ubuntu-latest
```

### jobs/tox.yml

This job template will install Python and invoke `tox`.

**Notes**:

- :warning: The latest `tox` release requires `python>=3.7`.

**Parameters**:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Optional. Defaults to `true`.
- `preCommit` (`boolean`): If set to `true`, this means [`pre-commit`] will be
  invoked inside a `tox` environment. Defaults to `false`. Optional.
- `pythonVersion` (`string`): The Python version to use for building and
  publishing the package. Defaults to `"3.11"`. Optional. Options: (`"2.7"`,
  and `"3.7"` through `"3.11"`).
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-20.04`]. Optional. Options: ([`ubuntu-20.04`],
  [`ubuntu-22.04`], [`ubuntu-latest`]).

**Example**:

```yml
stages:
  - stage: test
    jobs:
      - template: jobs/tox.yml@templates
        parameters:
          preCommit: true
```

[Github service connection]: https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#github-service-connection
[`local hooks`]: https://pre-commit.com/#repository-local-hooks
[`pre-commit`]: https://pre-commit.com/
[`ubuntu-20.04`]: https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2004-Readme.md
[`ubuntu-22.04`]: https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2204-Readme.md
[`ubuntu-latest`]: https://github.com/actions/runner-images/blob/main/images/linux/Ubuntu2204-Readme.md
