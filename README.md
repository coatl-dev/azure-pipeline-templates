# azure-pipeline-templates

[![Build Status](https://dev.azure.com/coatl-dev/azure-pipeline-templates/_apis/build/status/coatl-dev.azure-pipeline-templates?branchName=main)](https://dev.azure.com/coatl-dev/azure-pipeline-templates/_build/latest?definitionId=2&branchName=main)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/coatl-dev/azure-pipeline-templates/main.svg)](https://results.pre-commit.ci/latest/github/coatl-dev/azure-pipeline-templates/main)

## Usage

First, create a [GitHub service connection]. Then this to the beginning of your `azure-pipelines.yml`:

```yaml
resources:
  repositories:
    - repository: templates
      type: github
      name: coatl-dev/azure-pipeline-templates
      ref: refs/tags/v0.1.1
```

This will make the templates in this repository available in the `templates` namespace.

## Job templates

### jobs/pre-commit.yml

This template will install Python and invoke `pre-commit.

Parameters:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Defaults to `true`.
- `pythonVersion` (`string`): The Python version to use for building and
  publishing the package. Defaults to `"3.11"`. Optional. Options: (`"2.7"`,
  and `"3.6"` through `"3.11"`)
- `skipHooks`: A comma separated list of hook ids which will be disabled.
  Useful when your `pre-commit-config.yaml` file contains [`local hooks`].
  Optional. See: [Temporarily disabling hooks](https://pre-commit.com/#temporarily-disabling-hooks).
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-latest`].  Options: ([`ubuntu-20.04`], [`ubuntu-22.04`],
  [`ubuntu-latest`]).

Example:

```yaml
jobs:
  - template: jobs/pre-commit.yml@templates
    parameters:
      skipHooks: "pylint"
```

### jobs/publish-python-package-artifacts.yml

This template will allow you to upload your Python distribution packages in the
`dist/` directory to Azure Artifact feeds using `build` and `twine`.

Parameters:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Defaults to `true`.
- `pythonVersion` (`string`): The Python version to use for building and
  publishing the package. Defaults to `"3.11"`. Optional. Options: (`"2.7"`,
  and `"3.6"` through `"3.11"`)
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-latest`]. Optional. Options: ([`ubuntu-20.04`],
  [`ubuntu-22.04`], [`ubuntu-latest`]).

Example:

```yaml
stages:
  - stage: Deploy
    dependsOn: Test
    condition: contains(variables['Build.SourceBranch'], 'refs/tags/v')
    jobs:
      - template: jobs/publish-python-package-artifacts.yml@templates
        parameters:
          cache: false
          pythonVersion: "3.10"
```

### jobs/tox.yml

This job template will install Python and invoke `tox`.

Parameters:

- `cache` (`boolean`): If `true`, files and dependencies will be cached between
  pipeline runs. Defaults to `true`.
- `preCommit` (`boolean`): If set to `true`, this means [`pre-commit`] will be
  invoked inside a `tox` environment. Defaults to `false`. Optional.
- `pythonVersion` (`string`): The Python version to use for building and
  publishing the package. Defaults to `"3.11"`. Optional. Options: (`"2.7"`,
  and `"3.6"` through `"3.11"`)
- `vmImage` (`string`): Name of the VM Image to use for running the pipeline.
  Defaults to [`ubuntu-latest`]. Optional. Options: ([`ubuntu-20.04`],
  [`ubuntu-22.04`], [`ubuntu-latest`]).

Example:

```yaml
stages:
  - stage: Test
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
