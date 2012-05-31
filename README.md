Recovering from a broken Subversion-Git migration
-------------------------------------------------

It is not Subversion's fault, BTW.

1. Convert your Subversion repository to Git
2. Publish the result, announce it
3. Test it
4. Oops!
5. Recover
6. Test it
7. This writeup

A tool that we used to migrate the YaST repositories, had bugs, which resulted
in differences in content and properties of files.

Subversion repo, read only:

    o---o---o---o   trunk

Production repo:

    o---o---o---o---o---o   master (broken)
                ^       ^
                |       ` further development in git. contributors abound!
                ` subversion ends

Fixed repo:

    o---o---O---O   master (fixed)
                ^
                ` subversion ends

To complicate matters, have more branches than _master_ (you'd easily
guess that), about 7 of them. And we have not one repo but 150 of them. 

Assumption: one Subversion repo becomes multiple Git repos, spliting by module.

Requirements:

- git-core
- subversion, at least version 1.7
  so that `svn export` understands `--ignore-keywords`
- disk space: for *N* TODO copies of ...

recover-branch
==============

The `recover-branch` script takes these arguments:
1 broken and 2 fixed git repos of a subverison module
3 git branch name
4 svn url specific to the module and branch

- *Production Git URL*  (read-write access)
  TODO add a dry-run mode
  

- *Fixed Git URL*  (read-only access)
:
- git branch name
- *Subversion URL* (read-only access)
:  eg. file:///home/mvidner/2/yast2git/yast-svn

it
- sets up a Testing repo that has Production and Fixed as remotes

Testing:

    o---o---o---o---o---o   remotes/production/$BRANCH
        |       ^       ^
         \      |       ` further development in git. contributors abound!
          \     ` (subversion ends, unmarked in git)
           -O---O remotes/fixed/$BRANCH, $BRANCH

- compares the trees of Subversion and Testing (tip only)
- renames the branch refs for the Production remote:  (TODO tags)
  and explicitly marks where svn ended

Testing:

    o---o---o---o---o---o   broken/$BRANCH, $BRANCH
        |       ^         
         \      ` broken/svn/$BRANCH
          \     
           -O---O
                ^         
                ` svn/$BRANCH

- TODO do nothing if Fixed is a subset of Production

- rebase:

Testing:

    git rebase --onto svn/$BRANCH broken/svn/$BRANCH $BRANCH

    o---o---o---o---o---o   broken/$BRANCH
        |       ^         
         \      ` broken/svn/$BRANCH
          \     
           -O---O---o---o   $BRANCH
                ^         
                ` svn/$BRANCH

- TODO: how to push


- what users will see:

User's local repo, having developed on top of Production:

    o---o---o---o---o---o---o---o $BRANCH
                        ^         
                        ` remotes/origin/$BRANCH

The user will fetch and see the history changed (remotes/origin/$BRANCH moves):

    o---o---o---o---o---o---o---o $BRANCH
        |       ^       ^         
        |       |       ` remotes/origin/broken/$BRANCH
        |       |
         \      ` remotes/origin/broken/svn/$BRANCH
          \     
           -O---O---o---o   remotes/origin/$BRANCH
                ^         
                ` remotes/origin/svn/$BRANCH

...

recover-repo
============

`recover-repo` uses `recover-branch` for all branches and (optionally) pushes
the result to Production

- *branch map file*, optional:
  we start by looking at the git branches [older than the import date?].
  If master corresponds to trunk/$MODULE and the branches are all of the form
  branches/$BRANCH/$MODULE then no map is needed. Otherwise it defines the
   mapping of git branches to subversion branches(?), on lines of the form
  git_branch1 <whitespace> subversion_branch

recover-all-yast
================

`recover-all-yast` uses `recover-repo` on all the YaST repositories

