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

To work with repos with the sub-modules, do the following to clone

```bash
git clone URL-HERE
cd FOLDERNAME
git submodule init
git submodule update
```

Now, when the submodule needs to be updated, the simple way of doing that is by running the following in the project.

```bash
git pull --recurse-submodule

# Pull the changes and the new hash
git submodule update --remote NAME-OF-PROJECT-OR-FOLDER-NAME
git commit -am "Pull submodule update"
git push origin --all
```



