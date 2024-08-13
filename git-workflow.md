# Introduction

This document highlights my workflow when working with Git with a small team. There are prolly better ways to do this.

# Working on a project with a team

There are a few guidelines when working with a team and the style-guide and formatting and linting tools are essential. Then there is a what humans will do.

## Branches

Every GIT project will have branches. There are described as follows:

**Master/Main** : I still prefer master and this is where all the code lives. This is the backbone of my development and everything useful will be on this branch

**Release** : This branch is spun out of the master when things work. It contains released code that can be deployed. I prefer to add tags as well to mark these release versions.

**Development** : The development branch is a branch that runs in parallel with the master. It contains all things that may or may-not be tested/reliable. Thing of this as semi-tested stuff. Once it is stable, the branch will dip into the master branch to update the working code set. I like to add docs, comments, formatting and other bits before I merge to the master. Think of this branch is the ugly sister to the master.

**Feature/feature-name** : Feature branches are good. They should exist for no more than two weeks or a duration of a sprint. These will branch from development and then dip back.

**Bugfix/bug-id-in-tracker** : Same as above but just for bugs

**WIP.ipv1/something-long-term-experimental** : WIPs are work in progress and I add my initials to tell everyone that this is something that I worked on and will last longer than the two weeks. I might abandon it but there is value in pushing this to the repo.

## What to do as a developer

The first thing is to pickup a ticket or task. 

- Clone the repo
- Checkout Development
- Create a new branch with an appropriate name
- work on the code
- Commit often
- Once done and ready to test, git pull and merge the development into your branch to update it 
```bash
git fetch
# git rebase development
# or 
# git merge development

git add -A
git commit -m"Update branch from development"
```

Then push the branch to the repo. At this point, if you are using GitHub, create a pull request to merge origin/development from origin/new-task-branch-that-you-created.

## Git SubModules

Every now and then, I will create a library of code that is intended to be reused within two or mode projects. For this we use sub-modules. You will be able to see submodules with a hash as well as a .gitsubmodule folder.

### Adding a sub-module

A submodule is added simply by 
```bash
git submodule add URL-OF-GIT-REPO
git status
git add -A
git commit -m"Add sub-module XYZ"
```

### Cloning a project with sub-modules

Do the following to clone a project and then pull the sub-modules
```bash
# Clone the base repo
git clone URL-HERE

# Enter the project itself
cd FOLDERNAME

# Pull the sub-modules
git submodule init
git submodule update

# or replace the above two commands with one
git submodule update --init --recursive
```

Alternatively, run
```bash
# Replace all above commands with this.
git clone --recurse-submodules URL-OF-REPO
```

### Getting updates from the submodule

When the sub-project is updated, you need to get the updates rolled in. To do this, run the following.

Go to the sub-module folder and then run
```bash
git checkout development
git merge origin/development
```

Or 

```bash
git submodule update --remote
```

Now you need to update the container project by commiting the new sub-module code to it.

```bash
git diff --submodule
git commit -am"Update Sub-module from Remote"
git push origin --all
```

### Working on the submodule itself

When working on projects, we will be usually working on the sub-mmodule code itself and this is what is usually done where the sub-module is the code that needs to be updated.

```bash
cd sub-module-folder
git checkout development
git pull
```

Make changes and then create commits by running git add and then git commit

This is now ready to be updated to the repo
```bash
git submodule update --remote --merge
# or
# git submodule update --remote --rebase

# update container
git commit -am"Update submodule"
git push --recurse-submodules=check

# or
# git push --recurse-submodules=on-demand
# this will push the submodule before the container
```


