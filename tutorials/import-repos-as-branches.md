# Motivation

I wanted to tackle the following challenges:
1. easy discovery of all related repositories for developers
    * Usecase: in a microservice oriented architecture, developers can lose track of all the repositories that make up one giant software solution
    * Usecase: offering separate repositories for starter projects can become difficult to manage over time
2. unlimited repo usage (or at least feeling like its unlimited so people aren't hesitant to create more microservices)
    * The legacy github pricing plans with `limited repos but unlimited users` cost significantly less for some startups that cannot afford to bleed money when paying for `unlimited repos per user` under the new plans.

# Nay Sayers

Using submodules is possible when either your repos are public or you are ponying up sufficient cash for the private ones. This tutorial is focused on saving cash for when one needs to keep the code private.

# Proposed Solution

My co-worker [Sayan](https://github.com/tinker20) suggested the following:

```
Convert from:
> userName/repoNameABC
    > userName/repoNameABC/master
    > userName/repoNameABC/develop
    > userName/repoNameABC/feature/*
> userName/repoName123
    > ...
> userName/repoNameXYZ
    > ...

to this format instead:
> userName/masterRepoName
    > userName/masterRepoName/repoNameABC
        > userName/masterRepoName/repoNameABC/master
        > userName/masterRepoName/repoNameABC/develop
        > userName/masterRepoName/repoNameABC/feature/*
    > userName/masterRepoName/repoName123
        > ...
    > userName/masterRepoName/repoNameXYZ
        > ...
```

This would break:
* useful tools such as `gitflow`
    * unless they were forked and tweaked to match our needs

But it would work with:
* CI solutions which can trigger separate builds/projects based on wildcard expressions for branches

When I started wondering if anyone else was using this appoach, I realized that loopback was doing somethign somewhat similar but not as extreme:

```
> strongloop/loopback-example-database
    > strongloop/loopback-example-database/mysql
    > strongloop/loopback-example-database/oracle
    > strongloop/loopback-example-database/postgresql
```

With that precedent, I felt it was worth exploring. And the first real-world challenge was to import existing repositories as branches with their commit history intact, even though we would certainly lose the hashes.

# Steps

1. Decided to keep written track of my progress outside the repo until I had everything figured out otherwise I would have lost what I wrote down.
1. Created a multi-line command to help me start the experiment over & over again if I made mistakes (which I did many times):

    ```
    cd ~/dev && \
      rm -rf loopback-starter-projects && \
      mkdir ~/dev/loopback-starter-projects && \
      export PROJECT_ROOT=~/dev/loopback-starter-projects && \
      echo $PROJECT_ROOT && \
      cd $PROJECT_ROOT && \
      git init
    ```
1. In order to perfect a script for adding another repo as a branch, there were many crucial concepts to discover and understand before using them accurately:
    * creating an orphan branch
    * starting a branch from a specific commit point
    * auto-discovering fork-point or start-point for branches
    * cleaning up after switching between two branches that represent two completely different repos
    * running commands one at a time to double-check expected results before creatign a huge multi-line command
    * sometimes repos being imported can be strange or have mistakes like:
        * `develop` wasn't forked from `master`, or
        * a `feature` branch was forked from `master` instead of `develop`!
        * So it becomes important to carefully import them with human intervention to determine the appropriate `PARENT_BRANCH_NAME`
1. Import the `master`, `develop` and `feature/newrelic` branches of `pulkitsinghal/sample-notifier-service` repo (in that order):

    ```
    export REPO_URL_TO_IMPORT=https://github.com/pulkitsinghal/sample-notifier-service.git && \
      export REPO_NAME_TO_IMPORT=sample-notifier-service && \
      export BRANCH_TO_IMPORT=master && \
      git remote add repoToImport $REPO_URL_TO_IMPORT && \
      git fetch repoToImport && \
      git checkout --orphan $REPO_NAME_TO_IMPORT/$BRANCH_TO_IMPORT && \
      git reset && \
      git clean -nxd && \
      git clean -fxd && \
      touch dummy.txt && \
      git add . && \
      git commit -n -m "at least one commit must exist before importing another repository as a branch" && \
      git rebase repoToImport/$BRANCH_TO_IMPORT && \
    export BRANCH_TO_IMPORT=develop && \
      export PARENT_BRANCH_NAME=master && \
      export FORK_POINT=$(git merge-base --fork-point repoToImport/$PARENT_BRANCH_NAME repoToImport/$BRANCH_TO_IMPORT) && \
      echo "FORK_POINT: " $FORK_POINT && \
      git checkout -b $REPO_NAME_TO_IMPORT/$BRANCH_TO_IMPORT $FORK_POINT && \
      git rebase repoToImport/$BRANCH_TO_IMPORT && \
    export BRANCH_TO_IMPORT=feature/newrelic && \
      export PARENT_BRANCH_NAME=master && \
      export FORK_POINT=$(git merge-base --fork-point repoToImport/$PARENT_BRANCH_NAME repoToImport/$BRANCH_TO_IMPORT) && \
      echo "FORK_POINT: " $FORK_POINT && \
      git checkout -b $REPO_NAME_TO_IMPORT/$BRANCH_TO_IMPORT $FORK_POINT && \
      git rebase repoToImport/$BRANCH_TO_IMPORT && \
    git remote remove repoToImport
    ```
1. Import the `master`, `develop` and `feature/api-versioning` branches of `ShoppinPal/loopback-mongo-sandbox` repo (in that order):

    ```
    export REPO_URL_TO_IMPORT=https://github.com/ShoppinPal/loopback-mongo-sandbox.git && \
      export REPO_NAME_TO_IMPORT=loopback-mongo-sandbox && \
      export BRANCH_TO_IMPORT=master && \
      git remote add repoToImport $REPO_URL_TO_IMPORT && \
      git fetch repoToImport && \
      git checkout --orphan $REPO_NAME_TO_IMPORT/$BRANCH_TO_IMPORT && \
      git reset && \
      git clean -nxd && \
      git clean -fxd && \
      touch dummy.txt && \
      git add . && \
      git commit -n -m "at least one commit must exist before importing another repository as a branch" && \
      git rebase repoToImport/$BRANCH_TO_IMPORT && \
    export BRANCH_TO_IMPORT=develop && \
      export PARENT_BRANCH_NAME=master && \
      export FORK_POINT=$(git merge-base --fork-point repoToImport/$PARENT_BRANCH_NAME repoToImport/$BRANCH_TO_IMPORT) && \
      echo "FORK_POINT: " $FORK_POINT && \
      git checkout -b $REPO_NAME_TO_IMPORT/$BRANCH_TO_IMPORT $FORK_POINT && \
      git rebase repoToImport/$BRANCH_TO_IMPORT && \
    export BRANCH_TO_IMPORT=feature/api-versioning && \
      export PARENT_BRANCH_NAME=develop && \
      export FORK_POINT=$(git merge-base --fork-point repoToImport/$PARENT_BRANCH_NAME repoToImport/$BRANCH_TO_IMPORT) && \
      echo "FORK_POINT: " $FORK_POINT && \
      git checkout -b $REPO_NAME_TO_IMPORT/$BRANCH_TO_IMPORT $FORK_POINT && \
      git rebase repoToImport/$BRANCH_TO_IMPORT && \
    git remote remove repoToImport
    ```
1. Added the `README.md` and `tutorials/import-repos-as-branches.md` files, which were kept outside the project completely until now:

    ```
    git checkout --orphan master
    git reset && \
      git clean -nxd && \
      git clean -fxd
    mkdir $PROJECT_ROOT/tutorials
    touch $PROJECT_ROOT/README.md
    touch $PROJECT_ROOT/tutorials/import-repos-as-branches.md
    git add .
    git commit -n -m "initial commit"
    git remote add origin git@github.com:pulkitsinghal/loopback-starter-projects.git
    git push -u origin master
    ```

# References
1. Fork Point: https://git-scm.com/docs/git-merge-base#_discussion_on_fork_point_mode
1. Stack Overflow: Finding a branch point with Git?
    * https://stackoverflow.com/questions/1527234/finding-a-branch-point-with-git#answer-4991675
    * https://stackoverflow.com/questions/1527234/finding-a-branch-point-with-git#comment-30560790
        * https://stackoverflow.com/questions/1527234/finding-a-branch-point-with-git#comment-49931840
    * More than one way to figure out `FORK_POINT`:
        * Folks claim this way is more robust:

        ```
        export FORK_POINT=$(diff -u <(git rev-list --first-parent repoToImport/$BRANCH_TO_IMPORT) \
                                <(git rev-list --first-parent repoToImport/$PARENT_BRANCH_NAME) | \
                                sed -ne 's/^ //p' | head -1) && \
            echo "FORK_POINT: " $FORK_POINT
        ```
        * Out-of-the-box apporach but its effectiveness is contested:

        ```
        export FORK_POINT=$(git merge-base --fork-point repoToImport/$PARENT_BRANCH_NAME repoToImport/$BRANCH_TO_IMPORT) && \
            echo "FORK_POINT: " $FORK_POINT
        ```
1. [How to remove local (untracked) files from the current Git working tree?](https://stackoverflow.com/questions/61212/how-to-remove-local-untracked-files-from-the-current-git-working-tree)
1. Orphan Branches
    * https://git-scm.com/docs/git-checkout/1.7.3.1#git-checkout---orphan
    * https://stackoverflow.com/questions/12543055/how-to-push-new-branch-without-history
