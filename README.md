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

    o---o---o---o---o---o   Production/$BRANCH
        |       ^       ^
         \      |       ` further development in git. contributors abound!
          \     ` (subversion ends, unmerked in git)
           -O---O Fixed/$BRANCH

- marks where svn ended

Testing:

    o---o---o---o---o---o   Production/$BRANCH
        |       ^         
         \      |                                                         
          \     ` Production/svn/$BRANCH
           -O---O Fixed/$BRANCH, Fixed/svn/$BRANCH

- compares the trees of Subversion and Testing (tip only)
- sets Production/broken/$BRANCH = Production/$BRANCH 
- sets Production/$BRANCH        = Fixed/$BRANCH
- ifTODO do nothing if Fixed is a subset of Production

`recover-repo` uses `recover-branch` for all branches and (optionally) pushes
the result to Production

- *branch map file*, optional:
  we start by looking at the git branches [older than the import date?].
  If master corresponds to trunk/$MODULE and the branches are all of the form
  branches/$BRANCH/$MODULE then no map is needed. Otherwise it defines the
   mapping of git branches to subversion branches(?), on lines of the form
  git_branch1 <whitespace> subversion_branch

`recover-all-yast` uses `recover-repo` on all the YaST repositories

