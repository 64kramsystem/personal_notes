# Git

- [Git](#git)
  - [Document notes](#document-notes)
  - [General concepts](#general-concepts)
  - [Configuration](#configuration)
    - [Aliases](#aliases)
    - [Ignore commands/Gitignore](#ignore-commandsgitignore)
  - [Commit](#commit)
  - [Index operations](#index-operations)
  - [Log/Blame/Tree comparison](#logblametree-comparison)
    - [Formatting/Prettifications](#formattingprettifications)
    - [Pretty formats](#pretty-formats)
    - [Finding](#finding)
  - [Diffing/Patching/Status](#diffingpatchingstatus)
  - [Metadata](#metadata)
  - [Merging](#merging)
  - [Rebase](#rebase)
  - [Remotes](#remotes)
  - [Submodules](#submodules)
  - [Stash](#stash)
  - [Bisect](#bisect)
  - [Export (`archive`)](#export-archive)
  - [History rewrite](#history-rewrite)
    - [Remove a file](#remove-a-file)
    - [Remove a merge](#remove-a-merge)
    - [Batch operations (message/tree filtering, files removal)](#batch-operations-messagetree-filtering-files-removal)
  - [Useful operations](#useful-operations)
    - [Aggressive garbage collection](#aggressive-garbage-collection)
    - [Shell prompt](#shell-prompt)
    - [Correct whitespaces problems](#correct-whitespaces-problems)
    - [Revert/recover common mistakes](#revertrecover-common-mistakes)
    - [Find/set the default branch](#findset-the-default-branch)
    - [Checkout a GitHub PR (of a private repository)](#checkout-a-github-pr-of-a-private-repository)

## Document notes

The `git` command is implicit in all the examples, except where:

- it's a sequence of commands
- it's a single command with pipe

## General concepts

- `Target branch` : git merge $target_branch
- `First parent`  : (of a merge) the current branch when doing a merge
- `theirs`/`ours` : target/current branch, during a merge

Conventions:

- `$branch~[$n]` : reference to `$branch` minus `$n` (0 is also valid, and it refers to `$branch`)
- `branch^`      : `$branch` - 1
- `-$n`          : should be (HEAD - `$n`), but it's not the same, e.g. on merge commits
- `HEAD`         : head of the current branch (current commit)
- `ORIG_HEAD`    : last value of HEAD before dangerous operations
- `$ref@{-$n}`   : nth checkout previous to the last one; eg. checkout master => checkout feature => now `@{-1}` refers to master

## Configuration

Arbitrary sections/keys can be created, not only the ones used by Git.

```sh
config [--global] $section.$key $value # stores config in `.git/config`; [global] stores in `~/.gitconfig`
config --remove-section $section       # delete a section, with all the keys
help -c                                # list all possible configuration keys
```

| Parameter                 | Value (Suggested) | Comment                                            |
| ------------------------- | :---------------: | -------------------------------------------------- |
| `user.name`               |       name        |                                                    |
| `user.email`              |       email       |                                                    |
| `core.editor`             |      editor       |                                                    |
| `diff.tool`               |    merge tool     |                                                    |
| `difftool.merge_tool.cmd` |      cmdline      | example: "winmerge.sh \"$LOCAL\" \"$REMOTE\""      |
| `difftool.prompt`         |      `false`      |                                                    |
| `alias.alias`             |    'arguments'    | if arguments is only one, quotes can be omitted    |
| `branch.autosetuprebase`  |     `always`      | automatically rebase instead of merge when pulling |
| `diff.submodule`          |       `log`       | convenient submodules log display                  |
| `status.submodulesummary` |        `1`        | add module changes summary to `status`             |
| `push.recurseSubmodules`  |    `on-demand`    | automatically push submodules, if required         |
| `submodule.recurse`       |      `true`       | automatically updates the submodules               |


A section may be conditional, although this can also be accomplished per-project:

```
[includeIf "gitdir:~/work/"]
  path = ~/.work.gitconfig
```

The gitconfig search order is:

1. `/etc/gitconfig`
2. `~/.gitconfig` / `~/.config/git/config`
3. `$project/.git/config`

### Aliases

General notes:

- always surround complex aliases with double quotes; internal double quotes must be escaped
- the default shell is dash (sh)
- the current path is the project dir; see the `GIT_PREFIX` workaround below
- see `man git-config` for other infos

If an alias starts with `!`, it's a shell commmand. Use bash when:

- there are parameters involved;
- and/or dash is not sufficient.

```sh
# WATCH OUT!! TO VERIFY: Positional paramters must be *explicitly* referenced from position 0.
# Without `:0`, the first parameter specified is not included.
#
test = !bash -c 'echo "${@:0}"'

# Note that without `sh` as first parameter, the positional parameters don't behave as expected.
# IMPORTANT: doesn't work with filenames with spaces.
#
cs = "!bash -c 'cd \"${GIT_PREFIX:-.}\"; git checkout --$1 \"${@:2}\" && git add \"${@:2}\"' sh"

# Alternative form for complex commands (via function).
#
stsh = "!f() { git stash show -p stash@{${1-0}}; }; f"

# In order to autocomplete alias functions, on Zsh, use Zsh's autocomplete functionality
#
# Create the script `git_full_delete_branch`, then add the following; a function can't be used!
#
compdef _git git_full_delete_branch=git-branch
git config --global 'alias.brddx' '!git_full_delete_branch' # run this once
```

### Ignore commands/Gitignore

```sh
config --global core.excludesfile ~/.gitignore_global       # create a global gitignore

update-index --[no-]assume-unchanged $file                  # ignore changes to a file in the index
update-index --[no-]skip-worktree $file                     # ^^ same, but ignores more cases (eg. checkout)
echo '/.ruby-version' >> .git/info/exclude                  # ignore an untracked file only in the local repository
```

It's possible to ignore a directory also by adding a gitignore to it, containing `*`.

Ignore a directory but not a file levels below it; in order for this to work, all the directories in the middle must be excluded.

```
/application/**                           # this must end with `/**`
!/application/language/                   # files in this directory will not be included
!/application/language/gr/
!/application/language/gr/SomeFile.txt    # `/**` also works
```

For one level, it's easier:

```
/terraform/**/.terraform/plugins/*/*
!/terraform/**/.terraform/plugins/*/terraform-provider-archive_v*
```

## Commit

```sh
--no-commit                     	# doesn't commit for operations that does so by default (e.g. merge)

commit --amend [-m <newMessage>]  # amend the last commit; content of the index are added to the last commit; only [m]essage 

# Set the message/authorship/timestamp of a commit as another [retain timestamp on squash].
# Will also copy the message; [-c] prompts for the new message.
#
commit --amend -C $reference_commit
```

For coauthorship (multiple authors), see [here](github.md#coauthorship-multiple-authors).

## Index operations

```sh
add -i                          	# interactive adding
rm --cache					              # remove from the index (without deleting the file)

reset --(soft|hard) $commit     	# move the head pointer to the commit. if hard, the index is reset; if soft, the index will
                                  # contain the diff with the old data

git reset --soft $commit; git commit -c ORIG_HEAD    # amend a commit for more complex cases

clean [-d] [-f]                 	# remove untracked files; [d]irectories as well; [f]orce if option `clean.requireForce` is set

# perform reset HEAD and checkout of a file in staging; '--' is optional.
#
checkout HEAD -- $path

checkout -b $name $commit         # create branch from commit
```

## Log/Blame/Tree comparison

```sh
log --oneline --date-order --graph --all --decorate   # !! graph/diagram of the commits !!
script --return --quiet -c "git --no-pager log --no-color --graph --oneline" /dev/null > /path/to/out # dump the history to a file

log --branches=$branch $file        # search in another branch

log --merges --first-parent         # search only [merges] commits, only in the [first-parent] (eg. master when run from master)
log --no-walk --tags                # display only tagged commits

log -$n                             # show the latest $n commits, e.g. -1 for the last

# commits reachable by $r2 but not by $r1
# alias: `^$r1 $r2`
log $r1..$r2

# commits reachable by either $r1 or $r2, but not both
# alias: `$r1 $r2 --not $(git merge-base --all $r1 $r2)`
log $r1...$r2

log --diff-filter=[M|D] -- $file    # search for [M]odifications/[D]eletions of a file (`D` requires `--`)

log --grep=$regex                   # search by log message (case insensitive); simple regex, supports at least: '.*[]'; doesn't support: '+(){}'
log --author $email_pattern         # search by author; pattern is a substring, and also accepts `*`
log -S $string [$path]              # search the changes (additions/deletions) for $string, as string
log -G $regex [$path]               # search the changes (additions/deletions) for $string, as regex
log -- $path                        # search a deleted file (path is relative)

# Grep search (!); outputs with the grep format
# This has the advantage of (besides likely performance) searching only files in the index, which avoids cases like symlinks to unindexed files.
#
grep [--line-number] $regex
```

Blaming format:

```sh
blame [$branch] $file [-L $start[,$end]]                   # $end defaults to last line

git show $(git blame example.js -L 4,4 | awk '{print $1}') # show the commit of a certain line
```

Tree comparison:

```sh
cherry [-v] $upstream [$head]     # changes against tree (useful for rebasing: cherry master); [v]iew commit subjects
                                  # example commits in branch and not in master: cherry master branch
```

### Formatting/Prettifications

`--format` and `--pretty` are the same.

- `--pretty="format:"` : empty format, to avoid the cruft at the beginning of logs
- `--oneline`          : same as `--pretty='%h %s'`

### Pretty formats

- `%h` : abbreviated hash
- `%s` : subject first line (title)

### Finding

```sh
merge-base $A $B                                       # find common ancestor
show :/$regex                                          # show the last commit matching a regex in the message
branch --contains $commit                              # shows which branches contains the given commit
name-rev --name-only $commit                           # shows which tag the commit was in. '~N' indicates how many commits (N) before the tag
log --before="%F %R" $branch                           # show commits before/at given datetime
log --merges v0.1.8...v0.1.9                           # search merges between two tags

git rev-parse @{-1}                                    # find previously checkout out branch
git name-rev $(git rev-parse @{-1}) --name-only        # name of previously checkout out branch
[[ $(git cat-file -t $object 2> /dev/null) ]] && echo exists # check if a branch/commit/etc exists
```

## Diffing/Patching/Status

```sh
# `--color` applies to diff/show, and forces coloring (if `color.ui` != `always`); this may be undesirable for processing.
# `--ws-error-highlight=new,old` colors the removed trailing whitespace (which is not displayed by default (!!))

show rev[:file]                                 # diff rev^..rev
show -m $merge                                  # show the full diff of the parents of a merge (short form of `--diff-merges=m`)

diff [--cached]                                 # non committed files; [--cached] files in the index
diff $commit [$commit2]                         # can be used to show a merge diff!
difftool                                        # GUI version of diff
git difftool --extcmd='vim -d -c "windo set wrap" $5'    # Convenient vimdiff usage; requires `diffchar` plugin, otherwise, it's ugly

diff [--stat|--name-status|--name-only]         # only filenames with: stats+summary | status | nothing else
show <diff_options> [--pretty=""]               # same options as diff; --pretty hides heading information

status -sb                                      # more compact version of the status (show [b]ranch; first column: in index)

# Run a certain operation for each modified file.
#
git status -s | awk '{print $2}' | xargs -I {} scp {} myserver:mypath/{}
```

Diff/Patching:

```sh
# Patch with metadata retain (and commit).
# WATCH OUT!! The begin is the parent commit!
# $end_commit defaults to HEAD.
# Use `$parent_commit..$commit` to transfer a single commit.
# One patch per commit is created.
# `--reject`: in case of problems, partially apply the applicable patches, instead of not applying any
#
format-patch [--stdout] $start_commit~[..$end_commit]
am [--reject] $patchfile

# "Transfer" one or more commits between two repositories, while changing the files path.
# Don't forget the `g` regex modifier!
#
git -C ~/code/riscv_images format-patch --stdout $parent_start_commit[..$end_commit] |
  perl -pe 's| [ab]/ruby||g' |
  git am

# Patch with metadata loss (and no commit)
#
# WATCH OUT! One can configure `diff.noprefix true`, however, `format-patch` needs the prefixes to be
# specified in one way or another (they're mandatory, but `format-patch` has no configuration entries).
#
# - diff: suitable for standard patch import; with prefix, import using "patch -p1"
# - apply: `--check`: check instead of apply; `--reject`: see `am`
#
diff --no-prefix
apply [--check] [--reject] $patchfile
```

## Metadata

```sh
# Reference to upstream (remote) branch of HEAD. Returns a blank line if there isn't one.
#
for-each-ref --format='%(upstream:short)' $(git symbolic-ref -q HEAD)

# Repository name
#
ls-remote --get-url origin | ruby -ne 'print $_[/(\w+)\.git/, 1]'

# Find the branches/tags of a remote repo, without cloning (and print the last).
# Use `--tags --refs` to get the tags (see https://stackoverflow.com/a/12939216); can be combined with --heads.
#
git ls-remote --heads https://github.com/FFmpeg/FFmpeg.git | perl -lne 'print $1 if /refs\/\w+\/release\/([\d.]+)/' | sort -V | tail -n 1

# Find the tags reachable by the current head.
#
git log --simplify-by-decoration --pretty=format:'%D' | perl -lne '/^tag: (\S+)/ && print $1'

# Repository root path
#
rev-parse --show-toplevel

# Name of the current branch
#
rev-parse --abbrev-ref HEAD   # on non named branches (including remote ones) prints `HEAD`
symbolic-ref --short -q HEAD  # on non named branches prints nothing

# Git path relative to repository, from an absolute path. Note that the file must be in HEAD.
#
ls-tree --full-name --name-only HEAD $filename
```

Porcelain is supposed to be user-facing, however with `status`, it's for computer consumption; can be used to find the remote branch:

```sh
# The output listed is exact (but indented). Show [b]ranch info.

status -b --porcelain
  ## test...origin/test [gone]
  (files...)

status -b --porcelain
  ## master...origin/master
  (files...)
```

## Merging

```sh
checkout (--ours|--theirs) $files   # resolve conflict using ours/theirs version
merge -X ours|theirs                # solve merge conflicts using ours/theirs version
revert -m 1 $commit                 # revert a merge
merge --allow-unrelated-histories   # allow merging histories without a common ancestor!
```

Examples:

```sh
# Solve conflicts using always the local/remote copy (assumes no spaces in filenames).
# Use `--ours` (local) or `--theirs` (remote):
#
git st | awk '/both modified:/ { print $3 }' | xargs sh -c 'git checkout --theirs {} && git add {}'
```

## Rebase

```sh
rebase -i $parent_commit                               # interactive rebase from parent of given commit
rebase --onto $dest_branch $feat_branch_parent_commit

# Add a root (initial) commit
#
git commit --allow-empty -m "Initial (empty) commit"
git rebase -i --root

# Programmatically run an interactive rebase. Example: edit all the commits of a branch:
# WATCH OUT! The command runs on a file, so `-i` (in-place editing) is mandatory!
#
GIT_SEQUENCE_EDITOR="sed -i -re 's/^pick /e /'" git rebase -i "$(git merge-base HEAD master)"
```

It's possible to preserve merges using `--rebase-merges`, although it's not entirely clear how to do funky manipulations of history.

If a modification is applied to the history of a branch built on top of others and  `--update-refs` is specified (or the config `rebase.updateRefs` set), when rebasing interactively, the ancestor branches will also be updated!! In the interactive list, the references will show as `update-ref name_of_ancestor_branch`. WATCH OUT! This doesn't work the other way around (for descendant branches).

## Remotes

```sh
fetch [--all]                                            # fetch remote changes; [all] remotes
pull [--rebase]                                          # fetch+merge; do [rebase] instead of merging

remote                                                   # list the remotes; a remote is a label for a remote repos url
remote add $name $remote_url                             # add a remote (eg. for forks (upstream))
push [-u] [$remote [$branch[:$rem_branch]]]              # push changes; optionally only for a [branch] - if it's a branch, a remote branch is created; push
                                                         # set [u]pstream; set [rem_branch] if name differs from local
push --force-with-lease                                  # blocks overwriting if there are more commits on the remote branch
push --tags [--force]                                    # push [tags] information; this will *not* push the branch, unless also the branch data is specified ($remote $branch...)

push $remote :$branch                                    # delete a remote branch
push $remote :heads/$branch                              # delete a remote branch
push $remote :refs/tags/$tag                             # delete a remote tag

clone --depth 1 https://path/to/repo/foo.git -b $branch  # shallow clone (of a branch/tag)
```

Examples:

```sh
# List status of all the local branches.
#
for-each-ref --format="%(refname:short) %(upstream:track)" refs/heads
```

## Submodules

Submodules are tracked in a `.gitmodules` file.

```sh
# This takes care of adding the module, cloning and updating it, which otherwise must be done in steps.
clone --recurse-submodules $url

# Track a specific branch of the submodule!
config -f .gitmodules submodule.$sub_name.branch $branch_name

# Update all submodules (and init them, if empty).
submodule update --init --recursive

# Make pull recursively pull the submodules. As of v2.32, there is not config entry.
# WATCH OUT! `pull` alone won't update the submodules!
pull --recurse-submodules

# Update a single submodule
submodule update --remote $repo_name

# WATCH OUT! Must recurse the submodules (there is a config for this), otherwise, it's possible that
# the remote repo may refer to a submodule version that it doesn't hold!
push --recurse-subdmodules
```

Git treats the subdomule directory as special file (with special perm `160000`); its contents are not considered from outside it.

See [configuration](#configuration) for useful config entries.

## Stash

```sh
stash                         # save current changes to the stash
stash save [--patch]          # !! `--patch` = interactive stashing (can choose what to save) !!
stash (pop|apply)             # restore with/out clearing the entry
stash list
stash clear
stash show -p $stash_entry    # shows a stash entry content
```

## Bisect

```sh
bisect good $rev      # Mark an old enough rev as good: git automatically checks out the middle revision
bisect run $command   # Bisect using command exit code to find failures; if the command has aliases/etc, use `bash -c '<commands>'` as command.
```

## Export (`archive`)

```sh
fast-export --all      # export repo
fast-import            # import repo
archive $rev           # export a rev to stdout (defaults to tar format)

# Example export->import into another directory.
#
archive master Gemfile Gemfile.lock gems_dir | tar x -f - -C /export
```

## History rewrite

### Remove a file

In order to remove a file from the history, it's easier to use the [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner).

WATCH OUT! Don't forget to unprotect branches (e.g. `master`), otherwise, after pushing, the BFGRC will leave the repository in an inconsistent (history-wise) state.

### Remove a merge

To remove the merge `cleaner_pigz_compiling_flags`:

```
*   e1a9575 (HEAD -> master, origin/master) Merge branch 'increase_start_vm_timeout'
|\
| * f92c47a setup_system: Increase timeout when connecting to the VM
|/
*   8473f42 Merge branch 'cleaner_pigz_compiling_flags'
|\
| * 3b9f0df setup_system: Improve static compilation flags strategy
|/
*   fef2766 Merge branch 'add_qemu_log'
```

Remove the ones prefixed with `> `:

```
# Branch cleaner-pigz-compiling-flags
> reset onto
> pick 3b9f0df setup_system: Improve static compilation flags strategy
> label cleaner-pigz-compiling-flags

# Branch increase-start-vm-timeout
reset onto
> merge -C 8473f42 cleaner-pigz-compiling-flags # Merge branch 'cleaner_pigz_compiling_flags'
label branch-point
pick f92c47a setup_system: Increase timeout when connecting to the VM
label increase-start-vm-timeout
```

### Batch operations (message/tree filtering, files removal)

Mass-change all the messages in the current branch.

`--msg-filter` receives the commit message from stdin, and set as new message the output of the user-specified command.

Note that the [git filter-repo](https://github.com/newren/git-filter-repo) tool should be used instead. In order to take this approach, `export FILTER_BRANCH_SQUELCH_WARNING=1`.

```sh
# Rename commit titles.
git filter-branch --force --msg-filter 'perl -pe "s/^(\[Chef\] )?/[Chef] /i"' -- $(git merge-base master HEAD)..HEAD

# Delete a file
git filter-branch --force --tree-filter 'rm -f terraform/terraform.tfstate' master..HEAD

# Delete a Ruby method.
git filter-branch --force --tree-filter 'ag "def mymethod" -l | xargs -r perl -0777 -i -pe "s/^(\s+)def mymethod.*?^\g1end\n\n//sm"' master..HEAD
```

## Useful operations

### Aggressive garbage collection

```sh
git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

### Shell prompt

```sh
# Shows branch in the terminal
#
export PS1="\h:\W:\$(git branch 2>/dev/null | grep '^*' | colrm 1 2)\$ "
```

### Correct whitespaces problems

```sh
git add .
git diff --cached -w --no-color | git apply --cached -R
git commit -m 'Whitespace changes'
git commit -am 'All other changes'
```

Update git history, cleaning whitespace problems!:

```sh
rebase $base_commit --whitespace=fix
```

### Revert/recover common mistakes

See: https://sethrobertson.github.io/GitFixUm/fixup.html.

```sh
# Recover dropped stashes
#
gitk --all $(git fsck --no-reflog | awk '/dangling commit/ {print $3}')

# Recover lost commits.
# If not found in `--unreachable`, try `--lost-found`, which is subtly different.
#
git show $(git fsck --unreachable | git cat-file --batch-check | awk '/commit/ { print $3 }')
```

### Find/set the default branch

The rev-parse strategy works only for cloned repositories (see https://stackoverflow.com/q/17639383/210029), so if it's not present, we add it.

```sh
# Don't use `grep -q`, since it causes SIGPIPE.
#
if ! git branch --remotes | grep "^  $c_remote/HEAD " > /dev/null; then
  git remote set-head "$c_remote" -a > /dev/null # ignore noise
fi

git rev-parse --abbrev-ref "$c_remote/HEAD" | perl -ne "print /$c_remote\/(\w+)/" | tee >(xsel -ib)
```

Other:

```sh
# Online; always works.
#
git remote show $remote | awk '/^  HEAD branch:/ {print $NF}'

# Personal; use for special cases.
#
git config custom.development-branch devel
```

### Checkout a GitHub PR (of a private repository)

A PR request can be checked out locally (even if the source repo is private):

```sh
# This creates a standard branch, linked to the PR.
#
git fetch origin pull/$number/head:$local_branch
```
