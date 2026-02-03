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
We use the extremely fast [uv](https://docs.astral.sh/uv/) package and project manager for our dependency management. Splitting more complex monorepos into multiple packages, each dedicated to a specific sub-project, while maintaining common shared dependencies. Sub-project directories are defined as members of a shared [uv workspace](https://docs.astral.sh/uv/concepts/projects/workspaces/) as defined in the root level `pyproject.toml` file, along with the shared dependencies of the workspace.

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
It is the default behaviour of running `uv init` in a sub-directory to add the newly created package to the shared uv workspace in the root `pyproject.toml` file.

## The Issue - Dependency Conflicts
However, this approach does not scale well with increasing complexity, as sub-projects begin to have conflicting dependency requirements. Especially in our case where we have code from projects pushed to git over a period of multiple years and not necessarily maintained. This is what the uv docs have to say on the matter:

> Workspaces are intended to facilitate the development of multiple interconnected packages within a single repository. As a codebase grows in complexity, it can be helpful to split it into smaller, composable packages, each with their own dependencies and version constraints.

And:

> Workspaces are not suited for cases in which members have conflicting requirements, or desire a separate virtual environment for each member. In this case, path dependencies are often preferable.

## The Solution - Isolated Dependencies

### 1. Remove Root Level uv From Repo
If they exist remove the root level uv files and remove the , currently we are sharing the sub-projects dependencies as part of a shared workspace. Removing the root level uv files removes this 

### 2. Manage Dependencies Using Multi-Root VSCode Workspaces
By default, VSCode only scans the root directory of a project, meaning you have to manually add the interpreter path for each project sub-directory, open each project as an individual project or update the `$PATH` environment variable for VSCode to be able to use each projects `.venv`.

This can be overcome using VSCode multi-root workspaces, these allow the editing of multiple isolated projects in the same editor window. This is added to the repo by defining each project in a `.code-workspace` file in the repositories root `.vscode` directory.

Each sub-project is defined in a JSON format:

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

A `.vscode/settings.json` file is added to the root directory of each sub project with the below configuration telling VSCode to use the sub-projects root `.venv` as it's default Python environment.

```json
{
    "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python"
}
```

You can now launch the multi-root workspace in VSCode by navigating to `File > Open workspace from file...` and selecting the `.code-workspace` file in the repositories root `.vscode` directory. This configuration ensures that when you access files within a sub-project, or launch an integrated terminal, the sub projects environment is automatically loaded.