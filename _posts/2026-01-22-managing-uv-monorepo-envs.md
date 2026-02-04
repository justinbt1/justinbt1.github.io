---
title: 'Isolated uv Dependency Management in Monorepos'
date: 2026-01-22
permalink: /post/isolated_uv_dependency_management_in_monorepos
tags:
  - what I learned today
  - mlops
---
As a machine learning team we have a number of large projects that have their own repositories, but this is not practical for the large number of smaller, more data science focused projects we carry out. These smaller projects are stored within large thematic monorepos, with each project contained within its own subfolder.

```
|_monorepo/
  |_project_1/
  |   |_src
  |   |_notebooks
  |   |_models
  |   |_README.md
  |   |_pyproject.toml
  |
  |_ ...
  |
  |_project_n/
  |   |_src
  |   |_notebooks
  |   |_models
  |   |_README.md
  |   |_pyproject.toml
  |
  |_utils/
  |
  |_.gitignore
  |_README.md
  |_pyproject.toml
```
We use the extremely fast [uv](https://docs.astral.sh/uv/) package and project manager for our dependency management. Until recently we split more complex monorepos into multiple packages, each dedicated to a specific sub-project, while maintaining common shared dependencies. Sub-project directories are defined as members of a shared [uv workspace](https://docs.astral.sh/uv/concepts/projects/workspaces/) in the root level `pyproject.toml` file, along with the common shared dependencies of the workspace.

```toml
dependencies = [
  "pandas>=2.2.3",
  "tqdm>=4.67.1",
  "requests>=2.32.3",
  "bs4>=0.0.2",
  "boto3>=1.35.68",
  "lxml>=5.3.0",
  "polars>=1.30.0"
]

[tool.uv.workspace]
members = [
  "project_1",
  ...,
  "project_n"
]
```
The default behaviour of running `uv init` in a sub-directory, is to add a new `pyproject.toml` file to that directory and add the newly created package to the shared uv workspace in the root `pyproject.toml`.

## The Issue - Dependency Conflicts
However, the uv workspace approach does not scale well with increasing complexity, as sub-projects begin to have conflicting dependency requirements. Especially in cases where code from analytics projects is pushed to git over a period of multiple years and not necessarily maintained.

This is what the uv docs have to say on how to manage complex repositories:

> Workspaces are intended to facilitate the development of multiple interconnected packages within a single repository. As a codebase grows in complexity, it can be helpful to split it into smaller, composable packages, each with their own dependencies and version constraints.

And when using a uv workspace may not be the best approach:

> Workspaces are not suited for cases in which members have conflicting requirements, or desire a separate virtual environment for each member. In this case, path dependencies are often preferable.

## The Solution - Isolated Dependencies

### 1. Remove the root level uv definitions from repository
If they exist, remove any root level uv related files such as `uv.lock` and remove the root level `pyproject.toml` file. Each sub-project should still have their own directory, containing a 'pyproject.toml' file detailing that projects specific dependencies. The environment for a project can be created by navigating to that project directory using `cd` and running `uv sync`, this can be better managed using tools like VSCode's multi-root workspaces as detailed below.

It may also be useful to add the root level uv files to the repositories `.gitignore` file, to avoid any accidently created files being pushed to git. The uv specific additions to our `.gitignore` file are below, the `/` prefix ensures we only ignore root level files:

```shell
# monorepo management
/.python-version
/pyproject.toml
/uv.lock
```

### 2. Manage dependencies using multi-root VSCode workspaces
By default, VSCode only scans the root directory of a project, meaning you have to manually add the interpreter path for each project sub-directory, open each sub-project as an individual VSCode project in it's own window or update the `$PATH` environment variable for VSCode to be able to use each sub-projects isolated `.venv` environment.

Multi-root workspaces can streamline working with these isolated environments in VSCode, allowing for multiple isolated projects to be loaded in the same editor window with improved interpreter management.

A multi-root workspace can be configured by adding each sub-project to a `.code-workspace` file in the repositories root `.vscode` directory. This is a JSON format file which defines the sub-project directories to open as workspace members in a `"folders"` list. Each sub-project is represented by a dictionary containing the name of the sub-project and it's relative directory path. An example definition is below:

```json
{
	"folders": [
		{
			"name": "Project 1",
			"path": "../project_1"
		},
                ...,
		{
			"name": "Project n",
			"path": "../project_n",
		}
	]
}
```

To allow VSCode to automatically load the correct environment for each sub-project, a `.vscode/settings.json` file is added to the root directory of each sub-project. Each with the below parameters, which inform VSCode that the sub-projects root `.venv` environment should be loaded as that sub-projects default Python interpreter.

```json
{
    "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python"
}
```

You can now launch the multi-root workspace in VSCode by navigating to `File > Open workspace from file...` and selecting the `.code-workspace` file in the repositories root `.vscode` directory. This configuration ensures that when you access files within a sub-project, or launch an integrated terminal, the sub projects environment is automatically loaded.