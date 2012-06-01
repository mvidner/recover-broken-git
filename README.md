Recovering from a broken Subversion-Git migration
-------------------------------------------------

It is not Subversion's fault, BTW.

1) Code 2) Publish 3) Test.
Does not work if Publish operates on entire Git repos.

1. Convert your Subversion repository to Git
2. Publish the result, announce it
3. Test it
4. Oops!
5. Recover
6. Test it
7. This writeup

A tool that we used to migrate the YaST repositories, had bugs, which resulted
in differences in content and properties of files. (TODO links)

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
guess that), about 20 of them. And we have not one repo but 150 of them. 

Assumption: one Subversion repo becomes multiple Git repos, spliting by module.

Requirements:

- git-core
- subversion, at least version 1.7
  so that `svn export` understands `--ignore-keywords`
- disk space: for *N* TODO copies of ...

recover-repo
============

The `recover-repo` script takes these arguments:

1. *Production Git URL*
   (read-write access)
   TODO add a dry-run mode 

2. *Fixed Git URL*
   (read-only access)
   diverging from 1) in having actually correct contents

3. *Subversion URL*
   (read-only access)
   a root url (with all branches and modules)

4. *Subversion module name*
   corresponding to the git repos 1) and 2)

5. *branch map file*
   (TODO make it optional)
   Needed for cases where a git branch *b* corresponds to something else than
   the svn branch branches/*b*

What it does:

- sets up a Testing repo that
  is a bare clone of Production and
  has Fixed as a remote

- repeats the following for all git branches (TODO and tags, sans rebasing)

- finds and tags where the subversion part of the history ends

Testing:

    o---o---o---o---o---o   $BRANCH
        |       ^       ^
         \      |       ` further development in git. contributors abound!
          \     ` svn/$BRANCH
           -O---O remotes/fixed/$BRANCH

- compares the trees of Subversion and Testing (tip only)

* * *
(the code is tested up to here)
* * *

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


recover-all-yast
================

`recover-all-yast` uses `recover-repo` on all the YaST repositories

