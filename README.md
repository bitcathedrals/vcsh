# vcsh - git workflow in portable shell

vcsh is a set of git workflows bundled into a shell script. A shell
script is more  portable than almost anything else in the UNIX world.
The only real dependency other than a UNIX environment is git.

vcsh was originally patterned after git-flow.  Workflow tools take
procedures for using tools and put them into program rules. This
allows for trustability, repeatability, and confidence that the work
process will be simple and unremarkable.

My favorite little thing is you can show the commands without running
them. It is comforting when the tool is transparent. It gives you
confidence as you use it to get critical work done.

## high level view

Workflow tools automate a lot of drudgery repeated throughout the work
process. vcsh is able to call to git with multiple commands. The
commands are tailored to the workflow use case. Outputs are processed
and written to stdout. Errors, warnings, and any other noise is
written to stderr.

## The model

In the nitty gritty the model recognizes several standard branches.

- (main) -> main is the distribution branch, and the repository default.
- (develop) -> develop is a integration branch where development work is blended
            through publishing from feature branches to (develop) and pulling to
            feature branches from (develop).
- feature/(x) - feature branch (x) is where code is developed in isolation so the
                (develop) branch is kept in a clean, testable and bisectable state.
- bugfix/(x)  - a bugfix branch is mergeable into both (main) and (develop)

## Fundamental Ideas and Lexicon

The most important noun in vc is "root" and "branch". The branch is the
current git branch. The "root" is the trunk that the branch was derived
from. A heuristic is used to derive the root.

feature (branch) -> develop (trunk)
develop (branch) -> main (trunk)
main (branch) - > last tag with ^release (trunk)

This heuristic allows for the git commands to be supplied with branch
names automatically allowing the commands to be simplified, automatic,
and applicable across contexts.

### feature branches

Feature branches are where work originates. It has four primary
functions:

- development.
- publishing.
- integration. 
- exchanging code with peers. (regular git push to push)

```
get <remote> <branch>
get <none> 

get with two arguments, remote and branch, will checkout a local
branch from the "$remote/$branch". It will exit after that.

With no args it will require you to be in a feature branch,
it will look up the remote in the feature branch, perform a fetch
on the remote.

With commits from upstream it will attempt a fast forward only
remote, and if that fails ask to try a rebase.

This command is intended for collaboration on a feature branch,
to make getting changes added, via fast forward or rebase, easy,
and with minimal merges that produce a commit object.
```

```
integrate

integrate is an alias for rebase.

This command is designed to merge changes from the trunk into
the current feature branch.
```

rebase is built to auto detect root and branch. It does not
have a manual argument since it is tightlty integrated into
the model.

```
rebase

options: git options, arguments

Rebase the trunk onto the current branch. The trunk and branch values
are automatically deduced.
```

Rebase is flexible in that it works on the branches and trunks except
when the branch == main which is nonsensical.

```
dry run [rebase]: git log --date=relative --color main..develop | grep -v -E \'^$\'
dry run [rebase]: git rebase main
```

This is what the dry run looks like for a rebase. First it shows the commits
that would be rebased with -d switch for "dry run".

At this point we should introduce options which make commands very flexible.

## Options

vc options apply to the script, not the git commands. They handle
adjusting outcome and behavior to tune the debugging up.

```
vc options

Not all options are available for all commands, since some options
are nonsensible with the command.

vc options: vc options apply to most commands and provide very
            general commands like controlling output and
            enabling verbosity and debugging.

-v        : verbose
-d        : dry run
-t        : trace
-x        : debug

```

```
git arguments: arguments control the output of commands.

-compact      = show oneline compact form
-stat         = show diffstat output
-graph        = show graph of commit history
-details      = show branch details in log output
-decorate     = decorate history with branch information
-color        = color history output
-no-color     = don't color history output
-v            = verbose
<n>           = limit number of entries returned
-diff         = use a diff operation
-emacs        = use emacs as a difftool or mergetool
```

git arguments control the git output in various ways.

```
git options: tell vc what to do when generating git commands

:custom          = <branch>...?<trunk> or <trunk>...?<branch>
:upstream        = execute against upstream branch
:no-merges       = do not show merges
:only-merges     = only show merges
:merge           = use ... spec to show changes not in <trunk> and <branch>
:none            = internal, for tools that don't involve git version specifications like cat-file
```

All these can be used in combination except when they conflict or are
rejected by git which I would consider a bug.
