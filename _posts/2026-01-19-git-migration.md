---
title: 'Migrating Sub-Directory While Preserving Git History'
date: 2026-01-19
permalink: /post/migrating-subfolder-while-preserving-git-history
tags:
  - what I learned today
  - mlops
---

Recently we decided to open source some code we use internally for bibliometrics analysis, and share it in a public repository. However this code existed as a sub-folder in our analytics monorepo, so the question was how could we migrate the contents of that sub-folder to a new repo while preserving the git history for only those files?

It turns out this can be easily done using the `git-filter-repo` library, I've shared the steps below for anyone who needs to do the same.

### Prepare the Repositories
First create a completely empty repo on GitHub and clone it locally, then make a fresh clone of the repo containing the sub-folder of code to be migrated.

### Filter Repository Contents and Git History
To use the `git-filter-repo` library with Python, first you will need to install it into an active Python environment using pip or pipx:
```shell
pip install git-filter-repo
```
Then you can run the below command in the freshly cloned original repository to filter out all content from the repo and the git history, preserving only the history and files you wish to migrate:
```shell
git filter-repo --path path_to_subdirectory/
```
You may then want to tidy up the repository a bit, moving the sub-directory files to the top level of the repository, update or add a README, etc.

### Migrate to New Repository
The `filter-repo` command should have also removed the repositories origin from git, you can check this by running:
```shell
git remote -v
```
Once satisfied that the origin is no longer pointing at the original repository you can update it to point to the new empty one:
```shell
git remote add origin https://github.com/your_org/new_repo.git
```
You can then push the code to the new repostory:
```shell
git push origin main
```
  

*Hope this was helpful, or at least interesting!*
