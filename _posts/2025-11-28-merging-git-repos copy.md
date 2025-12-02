---
title: 'Merging Git Repos into an Existing Repository on GitHub'
date: 2025-11-28
permalink: /post/merging-git-repos-into-an-existing-repository-on-github
tags:
  - what I learned today
  - mlops
---
Recently we’ve launched a new project seeking to improve data quality of our impact reporting. This project has grown out of a number of smaller projects, some of which have their own git repository on GitHub. For project and code management reasons we have decided to combine these into a single monorepo, bringing all components of this project into a single place.

The challenge was that we wanted to:

- Merge multiple repositories into one that already existing repository.
- Preserve the Git history of the repositories we merge.
- Avoid merge conflicts with pre-existing monorepo code.
- Migrate open and closed issues to the new repository as we use GitHub Projects to manage our work.
- Avoid tightly coupling dependancies between projects.

In this article we go through how we merged each of these smaller project repositories into the main monorepo, including solves for the challenges we encountered.


## Monorepo Structure
The monorepo for our project is broken into directories each containing separate self contained sub-projects.

```
|_monorepo/
  |_project_component_1/
  |   |_src
  |   |_notebooks
  |   |_models
  |   |_README.md
  |   |_pyproject.toml
  |
  |_ ...
  |
  |_project_component_n/
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

Each projects is loosely coupled, with code that is mostly independent of the others, with some core shared utility code.

## Prepare for Merge
Before merging we needed to also prepare each of the project repositories to avoid issues with the merge. As an example the simplified general structure of one of our project repositories is:

```
|_project_repo/
  |_models/
  |_pipeline/
  |_notebooks/
  |_.gitignore
  |_README.md
  |_pyproject.toml
```

However if we attempt to merge this with our monorepo there will be multiple merge conflicts. To avoid this we moved everything for transfer to inside a new parent directory with the project name.

```
|_project_repo/
  |_project_name/
    |_models/
    |_pipeline/
    |_notebooks/
    |_README.md
    |_pyproject.toml
```

We also dropped the `.gitignore` as this already existed at the top level of the monorepo we hoped to merge into. This can all be achieved using the below commands within a **new branch** in the local copy of the project repository.

```sh
git branch -b prep_for_move
rm -f .gitignore
mkdir project_name
git mv -k * .* project_name
git add *
git commit -m "Prepare for move"
git push --set-upstream origin prep_for_move
```

We used `git mv` to avoid moving the `.git` directory from the top level of the local project repository. The flag `-k` avoids raising an error caused by trying to move the `project_name` directory to within itself.

## Merge the Repositories
Now for the part where we actually merged each individual project repo into the existing monorepo.

This was done by running the below commands from within the local git directory for the monorepo. First make sure that the local is set to the main branch of the monorepo, usually this is called main.

```sh
git checkout origin main
```

Then we needed to add the project repo as a remote on the local monorepo git repository. Make sure you replace `project_repo` with the name of the repo you wish to merge and `owner` with your GitHub account or org name.

```sh
git remote add -f project_repo https://github.com/owner/project_repo
```

Then merge the `prep_for_move` branch into the `main` monorepo branch. The `--allow-unrelated-histories` flag allows the merging of histories despite the repos having no common ancestor.

```sh
git merge -m "Merging project_repo" project_repo/prep_for_move --allow-unrelated-histories
```

We then removed the project repo as a remote repository to avoid any later complications.

```sh
git remote rm project_repo
```

Then finally we pushed the merged changes and histories to the main branch of the monorepo.

```sh
git push
```

Note: If there is a lot of code, which is likely given we are merging an entire repo, the push may time out. This can often be resolved by increasing the POST buffer size in the Git configuration.

```sh
git config http.postBuffer 524288000
```

## Bulk Migrate GitHub Issues
To bulk migrate GitHub issues from the project repo to the new monorepo, we used the `gh` GitHub CLI tool. See [here](https://cli.github.com/) for install instructions, you can then authenticate using `gh auth login` to connect to your GitHub account.

Then if you are in the local git directory for the project repo you can move (add to target repo, remove from project repo) the issues using the below command.

Though you will need to change `owner/monorepo` to match your target repo.

```sh
gh issue list --json number --state "all" | jq -c '.[].number' | while read issue; do
    echo "Trying to transfer issue number: $issue"
    gh issue transfer $issue owner/monorepo
    echo "Transfer successful"
done
```

What’s happening here:

- In the first section, the gh issue list — json number command retrieves a list of all issue numbers on repo_old as a JSON array. The option --state "all" means both open and closed issues are listed, to migrate only the active issues use --state "open".
- The next section uses the jq lightweight JSON processor to extract the number value and flatten the JSON into a flat bash array.
- Then this array is iterated over in a while loop, which uses the gh issue transfer command to transfer each issue based on it’s issue number to the target repo.

## Tidying Things Up
Finally we tidied up the project repository by deleting the temporary `prep_for_move` branch and then archived the repository.

*Hope this was helpful, or at least interesting!*
