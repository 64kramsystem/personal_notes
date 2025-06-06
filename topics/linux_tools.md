# Linux tools

- [Linux tools](#linux-tools)
  - [ls](#ls)
  - [find](#find)
    - [Examples](#examples)
      - [Search text inside multiple PDFs](#search-text-inside-multiple-pdfs)
  - [xargs](#xargs)
  - [tar](#tar)
  - [dar](#dar)
  - [cp/mv](#cpmv)
  - [mkfifo](#mkfifo)
  - [Files](#files)
    - [lsof](#lsof)
    - [Find/restore deleted but open files](#findrestore-deleted-but-open-files)
  - [PV (Pipe Viewer)](#pv-pipe-viewer)
  - [Parallel execution](#parallel-execution)
    - [Using GNU Parallel](#using-gnu-parallel)
    - [Using xargs](#using-xargs)
  - [Profiling (perf)](#profiling-perf)
    - [Waits on condvar](#waits-on-condvar)
    - [Sleep times](#sleep-times)
  - [GNU Screen](#gnu-screen)
  - [Partclone](#partclone)
  - [Clonezilla](#clonezilla)
  - [Dump CDs (safely)](#dump-cds-safely)
  - [Sleep](#sleep)
  - [Watch](#watch)
  - [Displaying messages](#displaying-messages)
  - [Nohup](#nohup)
  - [xdotool (X11 automation)](#xdotool-x11-automation)
  - [Dates](#dates)
    - [Formatting](#formatting)
    - [Operations](#operations)
    - [Calendar](#calendar)
  - [PDF/Images handling](#pdfimages-handling)
    - [Imagemagick (convert)](#imagemagick-convert)
    - [Raw to JPEG conversion](#raw-to-jpeg-conversion)
  - [Formatting/diff operations](#formattingdiff-operations)
  - [Encoding](#encoding)
  - [Benchmarking](#benchmarking)
  - [Images](#images)
    - [Mounting images](#mounting-images)
    - [Create an iso from a directory](#create-an-iso-from-a-directory)
  - [Create patches (diff)/restore them](#create-patches-diffrestore-them)
  - [Remote desktop](#remote-desktop)
  - [Docker](#docker)
  - [Wine](#wine)
  - [Build the Linux kernel](#build-the-linux-kernel)
    - [Building the Ubuntu mainline kernel](#building-the-ubuntu-mainline-kernel)
      - [Mainline configuration](#mainline-configuration)
    - [Other config notes](#other-config-notes)

## ls

```sh
ls -d               # don't [d]escend into directories
ls -D               # don't [D]ereference files

ls -r               # [r]everse order

ls -S               # sort by [S]ize (default: desc.)
ls -t               # sort by m[t]ime (default: desc.)
ls -v               # sort numerically (!)

ls --block-size=K   # human-readable; use K,M,G...
```

## find

Note:

- file traversal ordering is not guaranteed
- the order is crucial!, e.g. `-print` must go after `-name` etc.

```sh
! (condition)                     # negates a condition
-name xx -not -name yy            # negate search condition
-name "*.c" -o -name "*.h"        # simple OR condition
\( -name xx -or -name yy \)       # OR condition with brackets

-[i]name <pattern>                # use quotes when using wildcards in pattern; case [i]nsensitive
-wholename <value>                # include the directory name (eg. `.git/config`)
-regextype egrep -[i]regex <regex>  # same as name, but with regex; must match the whole name (including the path, if any)
                                    # WATCH OUT: must specify the regextype (the default is minimal); if there is a -not, it must be put before it
                                    # see section below for regex type notes.

-type (f|d)                       # [f]iles, [d]irectories
-type l                           # find symlinks, not recursively following
-L $path -xtype l                 # find symlinks, recursively following
-L $path -type l                  # find broken symlinks, recursively following

-maxdepth <value>                 # max number of levels to descend; `1` causes not to descend into directories
-mindepth <value>                 # opposite of `maxdepth`; useful when the root path (eg. `.`) must not be displayed (use `1`)
-follow                           # [follow] symlinks

-perm                                    # parameters can be either in number or literal ("u+w,g+w","o=w") format
      <value>                            # find where permissions are exactly <value>
      -<value>                           # find where all <value> permission are set
      /<value>                           # find where any of <value> permissions are set
-user <user>                      # [user]

-(a|c|m)[-|+](time|min)           # Accessed|Changed|Modified; time = n*hours, min = n*mins; +/- = more/less than
--mtime +7                        # mtime = file data last modified N*24 hours ago; +7 = modified more than 7 days ago
--mtime 0                         # 0 = created today
--mmin -<minutes>                 # mmin = modified in the last <minutes> minutes (don't forget the minus)

-size [+|-]<value>[c|k|M]         # bigger or greater than <value>; c/k/M = bytes/kilo/mega
-empty                            # empty files/directories

-not -path '*/.*'                 # skip hidden files (see https://askubuntu.com/q/266179)

-print                            # when passing the content to the pipe, adds the filename to the start of every line
-print0 | xargs -0                # use null-char to separate filenames (useful for filenames with spaces)
-print -quit                      # print one entry and quit; useful to check if there is one or more files in a directory (or matching)
-printf '%P\0'                    # print the relative path, and also append null-char

# exec: in both, `{}` must be the last argument; if not possible, use shell (see below)
-exec $command {} \;              # execute command for every file found, passing the name as {}; multiple -exec can be passed
-exec $command {} +               # execute command with as many files as possible (see https://unix.stackexchange.com/a/389706)
```

Printf params:

- `%f`: basename
- `%h`: parent directory (!!!)

Regexp types:

- `egrep`
  - doesn't support: `\d`
  - supports:        `\w`, `\b`, `[:digit:]`

### Examples

Simplest find files + execute command (with quoting):

```sh
# Without xargs
find . -type f -exec bash -c 'mv "$@" /dest' - {} +

# With xargs
find . -type f -print0 | xargs -0 -I {} mv {} /dest
```

Execute a certain command for the found directories:

```sh
# Better approach. The null-char approach is needed in order to cover filenames
# with double quotes.
#
find . -name no -print0 | xargs -0 -n 1 bash -c 'mv "$1" ../exclude/"$(basename "$(dirname "$1")")"' {}

# Simpler, but worse approach - fails if there are double quotes in the directory name.
#
find . -mindepth 1 -type d -exec bash -c 'cd "{}" && pwd' \;
```

Excluding paths in find (see https://stackoverflow.com/questions/4210042/exclude-directory-from-find-command).
Note that the -path must be relative to the same path as the search path, and it also must end with '/*'.

```sh
find . -name 1.txt -not -path ./skipdir/* -not -path '*/internal_dir/*'
```

Cycle filenames, with whitespace support.

```sh
# WATCH OUT! "<<<" won't work, because Bash parameter substitution doesn't support null bytes
#
find . -name "*.txt" -print0 | while read -rd $'\0' filename; do
  echo "<$filename>"
done

# If the cycle needs to break on the first find, ie. in case of particular conditions, and it's in
# a function, then it needs special handling, as `return` won't work, because it's in a subshell:
#
function myfunc() {
  local result

  result=$(
    find . -name "*.txt" -print0 | while read -rd $'\0' filename; do
      if [[ $filename == "matching.txt" ]]; then
        echo "$filename"
        break
      fi
    done
  )

  if [[ $result != "" ]]; then
    return
  # etc.etc.
}
```

Test if a directory contains directories:

```sh
! find -mindepth 1 -type d -exec false {} +
```

Execute a command for the content of a directory; using `for f in $dir/*` works, however, if there are no files, it raises an error.

```sh
find $dir -name $glob -exec bash -c '
  zpool labelclear -f "$1" 2> /dev/null || true
' _ {} \;
```

Sort files by modification time (simplified; assumes no newlines):

```sh
find . -printf '%T@ %p\n' | sort -n | sed -z 's/.+? //'
```

#### Search text inside multiple PDFs

```sh
find . -name '*.pdf' -exec sh -c 'pdftotext "{}" - | grep -i --with-filename --label="{}" --color "<pattern>"' \;
```

## xargs

Note that, when grouping args, they must be at the end of the command (with -I, one invocation per arg is performed). Empty lines are ignored.

For the parallel concepts, see the specific section.

```sh
-0                                    # specify null-char separator; useful ie. with `-print0`
-r --no-run-if-empty                  # :-)
-L $lines --max-lines=$lines          # maximum input lines per command
[-I pattern]                          # cmd split input on new lines and executes cmd; use of -I is the simplest
                                      # if no args, appends the line at the end
-d '<delimiter>'                      # set delimiter, e.g. "\n" with `ls`

```

Examples:

```sh
ls -1rt | xargs -d "\n" $cmd                  # executes the command, with files sorted by reverse modification time
cat /proc/self/environ | xargs -0 -L 1        # print the env vars (null-terminated), one per line
```

## tar

WATCH OUT!! When extracting files whose parent directories were not explicitly specified during archival, the owner will be root (see example at the bottom).

```sh
tar -C /tmp xvz                                        # Extract to a different directory
tar --exclude='parsec-benchmark/.git' parsec-benchmark # Exclude (glob pattern) (also on decompression); if `/` is not prefixed, matches at any level
tar xv -O -f "$tarfile" metadata.gz | gunzip > "$outfile" # Extract a single file to stdout (and redirect to a file)

# The BSD tar can extract zip files from the pipe!!
#
curl http://x.co/bar.zip | bsdtar -xf -

# Rename destination files while extracting!!
#
# - Basic sed regex!!
#   - Not supported (at least): +, \d, \K, capturing groups, ? (greedy)
#   - Supported: ^, $, \b, [], regex delimiters
# - The regex applies to the WHOLE name, so be careful when changing dir names
# - WATCH OUT!! Directory names should not have a trailing slash, otherwise the files are transformed,
#   but not the root dir; in order to avoid ambiguity with dirs sharing the same prefix, a solution is
#   to use `\b` where applicable.
#   - Interestingly, when transforming a dir, if the trailing slash is used, the target dir is displayed
#     (with --show-transformed-names) with the trailing slash, but it's not transformed
# - Untransformed names are displayed by default
#
tar xvz --transform="s|^parsec-3.0|parsec-benchmark|" --show-transformed-names

# Create an empty archive
#
echo -n | tar c --files-from -

# Use -C and wildcards at the same time. See other alternatives at https://superuser.com/q/266422.
#
$ (cd /path && tar cf *.xml)
```

Intermediate directories ownership problem. There isn't any trivial solution; the simplest way is to split into multiple archives by owner.

```sh
mkdir -p foo/bar
sudo tar c foo/bar > foo.tar  # sudo dar -c foo -g foo/bar
rm -rf foo
sudo tar -x -f foo.tar        # sudo dar -x foo
# root!
ls -ld foo
```

## dar

**WATCH OUT** dar is much slower than tar, in any case:

- without compression
- with compression
  - additionally, CPU usage is poor compared to `tar | zstd -T0`
- in WSL, extracting an archive with lots of files from a flash drive, caused a long delay at the beginning

```sh
# Include only some files, in the current dir. The slice suffix (.<n>.dar) can't be avoided.
#
# -vt -vs      : progress, only included/excluded files
# -w|--no-warn : don't warn on overwrite
# -z -G        : use zstd (level 3) (don't put a space), 32 threads (can't use for streaming)
#
# WATCH OUT! If interrupted, dar leaves a partial archive.
#
dar \
  -vt -vs -w \
  -zzstd:3 -G 32 \
  -c archive \
  -g zeus.json -g Gemfile

# Set root dir; exclude files with different logic.
#
dar -c archive \
  -R / \
  -P "etc/netplan/01-network-manager-all.yaml" \
  -P "saverio/.local/share/data/Mega Limited/MEGAsync/*.socket" \
  -P "**/target"

# Display progress (!!).
# Ordering of the find options is meaningful; don't change.
#
files_count=$(
  find /source/projects /source/other_projects \
    \( -path "/source/projects/linux-*" -o -path "*/target" \) -prune \
    -o -print \
  | wc -l
)
dar -vt \
  -c /dest/myprojects \
  -R /source \
  -g projects -g other_projects \
  -P "projects/linux-*" -P "**/target" \
| grep "^Adding file to archive" \
| pv -l -s "$files_count" > /dev/null

# Extract archive.
#
# -vt      : progress (not default)
#
# In order to display progress, use (`| grep "Restoring file's data" | pv ...`).
#
# IMPORTANT! dar restores the intermediate directories original owner, differently from tar.
#
sudo dar -vt -x archive -R /path/to/dest

# Create an archive with (par2) error recovery.
# The recovery data is added separetely, so it's not efficient.
#
dar -c archive `#blah...` par2

# List contents
#
dar -l archive
```

## cp/mv

In order to copy/move the content of a directory including hidden files, use:

```sh
# `-T`(--no-target-directory): will create the dest dir if it not exists; if it exists, it will copy
#     inside it. Without this, if dest dir exists, `source` will be created inside it.
#
# source can include a trailing slash
#
cp -rT /source /dest

# Also includes files starting with double dot (!); WATCH OUT! it errors if there are no files corresponding to each pattern.
# Alternative: use the `dotglob` shopt.
#
mv source/{,.[!.],..?}* dest
```

## mkfifo

Messages don't need termination; this is abstracted, so empty messages and binary files can be sent:

```sh
# Reads and writes block until there is a write/read on the other end.
#
mkfifo /tmp/fifo

echo -n > /tmp/fifo &

echo "<$(cat /tmp/fifo)>"
# <>
```

## Files

```sh
mktemp --suffix="${filename##*.}"            # Create a temporary file, using the extension of $filename
mktemp --tmpdir=$parent_dir                  # Use $parent_dir as parent (doesn't create it); WATCH OUT!! `=` is required after `--tmpdir`
mktemp [-d] "${TMPDIR:-/tmp}/myprefix-XXXXX" # Mac-compatible (opt. dir), without using `--suffix`; WATCH OUT!! the X's must not be followed by any char
                                             # WATCH OUT!! don't append anything after `-XXXXX`
stat $filename --format='%s'                 # Get file size
```

Making a virtual file from the concatenation of multiple files:

- Solutions using standard tools have limits, e.g. loop+mdadm may require n*512 byte files', nbd didn't work.
- References:
  - https://unix.stackexchange.com/questions/94041/a-virtual-file-containing-the-concatenation-of-other-files
  - https://serverfault.com/questions/487670/concatenating-files-to-a-virtual-file-on-linux
- Working solution: https://github.com/schlaile/concatfs

### lsof

Funky lsof stuff; note that a program may not always keep the files open while is running!

```sh
lsof $filename            # specific open file[s]; also works on mountpoints
lsof +D $directory/       # files open in a specific directory (end slash required!); also works on mountpoints
lsof -c $process_prefix   # files opened by a process starting whose name starts with a given prefix
lsof -p $pid              # files opened by a process with the a given pid
lsof -i :$port            # processes listening on a given port
```

### Find/restore deleted but open files

```sh
lsof -s | grep deleted

lsof | grep fileName					     # get the process id (col 2) and the file descriptor (col 4, remove the letter)
cp /proc/$pid/fd/$fd $dest         # recover the file
cat /dev/null > /proc/$pid/fd/$fd  # truncate a deleted open file!!
```

## PV (Pipe Viewer)

Run multiple instances:

```sh
# [c]ursor, [N]ame
#
pv -cN in foo | gzip | pv -cN out > bar
```

## Parallel execution

### Using GNU Parallel

In general, GNU Parallel has some convenient functionalities, compared to xargs.

The number of jobs is automatically limited to the number of cores.
Quoting is not required. The shell used when running a string is the current one.
The output is displayed serialized, but the execution is parallel!

```sh
ls -1 *.tar.* | parallel tar xvf      # if unspecified, the argument is automatically appended
ls -1 | parallel zip -m {1}.zip {1}   # parameterized; can also use `{}`
```

Using the `:::` commands separator:

```
parallel echo ::: A B                                              # in parallel: (echo A) (echo B)
parallel echo vmsav{1} ::: A B                                     # in parallel: (echo vmsavA) (echo vmsavB)
parallel --xapply echo "{1} {2}" ::: "${arr1[@]}" ::: "${arr2[@]}" # zip two arrays (`--xapply`)
```

Examples:

```sh
# Placeholder manipulation, e.g. to change the output path!
#
find /path/to -type f | parallel "ffmpeg -i {} /path/new/{/}"

# Multiple commands for each job
# Redirections are supported.
#
ls -1 | parallel "ffmpeg -i {} {}.wav && neroAacEnc -q 0.5 -if {}.wav -of {}.m4a"

# Receive multiple arguments per process.
# `--plus` activates many features, including positional parameters.
# The `:::` is a syntax to pass parameters; `:::+` specified extra parameters.
#
parallel --plus "echo {1}/{2}" ::: l1p1 l2p1 :::+ l1p2 l2p2
```

Other options:

- `-u|--ungroup`          : don't group output (print lines immediately)
- `--delimiter $delim`    : input delimiter; WARNING! Newline is still considered a delimiter
- `-P|--max-procs <N>[%]` : max procs; `%` (optional) accepts decimal; watch out! uses the ceiling (e.g.13% of 32 yields 5)

In order to programmatically confirm the citation, must run as `echo "will cite" | parallel --citation || true`.

### Using xargs

WATCH OUT! Differently from GNU Parallel, the command can't be quoted - use `bash -c` for that.

- `-P <processes>`: use 0 for unlimited; based on the manpage, `-n` or `-L` should be used, but they weren't required with the personal use cases.

```sh
# WATCH OUT! When sending params via echo on a single line, must not send the newline (which is always considere a separator):
#
echo -n t1 t2 t3 | xargs -d ' ' -P 0 -I {} echo mysql -e 'SELECT * FROM {}'

# Execute a command from `ls` output.
#
ls -1 *.txt | perl -pe 's/\.txt//' | xargs -P 4 -I {} mysql -e 'LOAD DATA INFILE "{}.txt" INTO TABLE "{}"'

# Prints the files, surrounded by double quotes.
# Double quotes needs to be escaped! Additionally, must escape the backslash, otherwise it's interpreted by Perl.
#
ls -1 *.txt | perl -pe 's/(.*)/\\"$1\\"/' | xargs -P 0 -I {} echo {}

# Placeholder manipulation is not directly supported, but can use shell functionalities.
#
find /path/to -type f | xargs -I {} sh -c 'ffmpeg -i "$1" "/path/new/$(basename "$1")"' _ {}
```

Ignore failing commands:

```sh
seq 4 | xargs -I {} -P 0 sh -c 'aws ec2 delete-snapshot --snapshot-id {} || true'
```

## Profiling (perf)

Perf is bound to the given kernel version; when this conditions can't be fulfilled:

- build it (https://unix.stackexchange.com/a/530898; requires `libelf-dev libdw-dev`): `make -C tools/ perf_install prefix=/my/path`
- can directly invoke the binary (hack): `/usr/lib/linux-tools/5.15.0-41-generic/perf`

See install script for convenient tweaks.

```sh
# Display stats (to stderr!!)
#
# -e                   : events recorded
# -x|--field-separator : print in tabular format, with given separator
# -p                   : attach to process
#
perf stat -e L1-dcache-load-misses,context-switches,migrations,cycles,sched:sched_switch --per-thread -x, -p $(pgrep -f qemu-sys) 2>&1 | tee perf.txt

# Record in-depth profile (to `perf.data`).
#
# This can take lots of space; recording 64 threads for around a minute created more than 1 GB of data.
#
# -e, -s, -p         : same as `stat`
# -s|--stat          : record per-thread event counts (NOT --per-thread!!!)
# -c|--count         : event period
# -g                 : enable call graph
# --call-graph dwarf : use when `-g` doesn't work (see end of this section); implies `-g`
# -o $file           : specify output file
# -F|--freq $hz      : specify the frequency (default=1000)
#
perf record -e sched:sched_stat_sleep,sched:sched_switch,sched:sched_process_exit -s --call-graph dwarf -p $(pgrep -f qemu-sys)

# Display the record results; includes the call graph.
# It's not clear why the "PID TID" section at the end is empty.
#
# -T|--thread     : show per-thread (requires `record -s`); see `prev_pid=$tid` entries
# --tid           : show only a single tid
# --stdio         : text output (instead of GUI)
# --no-call-graph : display in tabular format, without call graph
# -n              : add samples; doesn't seem to be particularly useful.
# --sort comm     : sort by command (easiest to read)
#
perf report --thread --stdio
perf report --stdio --no-call-graph --tid=mytid --sort comm

# Convenient display; uses `inferno` crate.
# Possibly, can use `https://github.com/flamegraph-rs/flamegraph`.
#
perf script | inferno-collapse-perf | inferno-flamegraph > rustc.svg

# Terminate, with data saving.
#
kill -INT $(pgrep perf)
```

General information:

- `record` and `stat` can yield similar results, but they can't be used interchangeably (see https://stackoverflow.com/questions/49216628/perf-stat-vs-perf-record).
- `$event:u` monitors only at user-level privilege; in some cases, kernel-level monitoring is important

Record/report:

- generally speaking, if the call graph is insufficient:
  - enable the debug information for the given program (e.g. `--enable-debug-info` for QEMU)
  - disable optimizations: `-Og` or `-O0`
    - `-Og` adds extra info, but also treats more warnings as errors (use if certain that it's ok `-Wno-error`)
    - on paper, `-O0` produces less debug info, but with QEMU, it produces more
  - set `-fno-omit-frame-pointer`
  - macros seem not to affect the production of call stacks
- `--call-graph dwarf` can get more information than the default, however, it doesn't help anyway with optimizations/lack of debug info
  - it can take 100x as much space (!!)

Events:

- `cpu-cycles` (synonym of `cycles`) is better choice than `cpu-clock`; use the latter only if the first can't be used

### Waits on condvar

Requires recording `sched:sched_switch`; can also try `sched:*switch` for all the context switches.

```sh
# Convenient statement for checking the context switches caused by a thread waiting on a condvar.
#
grep __pthread_cond_wait $report_output -w -B10 | grep prev_comm=qemu | sed 's/%.*//' | awk '{i+=$1} END {printf("%.2f%%\n",i)}'
```

Reference: https://easyperf.net/blog/2019/10/12/MT-Perf-Analysis-part2#find-expensive-synchronization-with-perf-

### Sleep times

Requires recording `sched:sched_stat_sleep,sched:sched_switch,sched:sched_process_exit`.

WATCH OUT! For unclear reasons, it fails, even with the official [patch code](https://lwn.net/Articles/510120).

```sh
# -v|--verbose
# -s|--stat    : special mode to merge sched events in order to perform sleep analysis
#
perf inject -v -s -i perf.data -o perf.sleep.data
perf report --stdio --show-total-period -i perf.sleep.data
```

Reference: https://perf.wiki.kernel.org/index.php/Tutorial#Profiling_sleep_times

In order to use the LWN example, add:

```c
#include <time.h>
#include <sys/time.h>

struct timespec ts1;
struct timeval tv1;
```

## GNU Screen

Commandline params:

- `-ls`                : list session
- `-r [$name]`         : resume screen session; if an exact match is not found, prefix search is performed (but must match only one)
- `-x [$name]`         : join named session (see `-r` for matching rules)
- `-h $scrollback`     : size of scrollback buffer
- `-dm`                : (special) start session in detached mode
- `-S $name`           : session name
- `-X $command`        : execute a command in the session (see example below)
- `-L`                 : enable logging (flushes every 10" by default); doesn't overwrite existing files
- `-Logfile $filename` : set logfile (does _not_ imply `-L`)

Commands:

```
Ctrl+a                             function shortcut
  0..9                             go to window
  A                                set name
  K                                kill
  c                                create new
  C (:clear)                       clear screen. NOTE: in order to clear the scrollback buffer, one needs to set it to zero, then back to the original size
  d                                detach
  "                                list of windows
  h (:hardcopy)                    dump (hardcopy) screen in file
    -h $filename                   dump entire buffer, not only visible screen
  :                                command mode

Copy-related:
  [                                copy mode; lets the user scroll (Shift+PgUp/Down) and mark paste buffer start/end (using space)
  scrollback lines                 set the scrollback buffer; default=5000
  >                                output paste buffer to file

defscrollback $lines               set scrollback buffer in .screenrc
logfile flush $seconds             change the flush period (nonnegative integer; defaults to 10)
```

Execute commands in a detached session:

```sh
# `stuff` is the command; the terminating newline is necessary.
#
screen -dmS $session_name                  # start a named session (-S $name) in detached mode (-dm)
screen -r $session_name -X logfile flush 0
screen -r $session_name -X stuff 'ls^M'    # ^M is enter

# run a command (then wait) in a new session; supports detached session.
#
screen bash -c 'mycommand -foo; read -rsn1'
```

In some workflows, it's convenient to set `IGNOREEOF`, in order to prevent accidental exits from the screen session.

## Partclone

```sh
# Backup
partclone -c -s /dev/sdX | pzstd > /path/to/dumpfile

# Restore
zstd -d /path/to/dumpfile | partclone -r -o /dev/sdX
```

TO REVIEW:

```sh
# Dump a single partition ######################################################

# Find the partition type
#
mount /dev/sdc1
df -T /dev/sdc1
umount /dev/sdc1

# [c]lone, [d]ebug info, [s]ource, [o]utput
#
partclone.vfat -c -d -s /dev/sdc1 -o - | pixz -2 > sdc1.vfat-ptcl-img.xz.aa

# Restore a single partition ###################################################

# [d]ebug info, [s]ource, [o]utput
#
gunzip -c pizza.gz.a* | partclone.restore -d -s - -o /dev/sdc1
```

## Clonezilla

Setup:

```sh
# Setup, from 20.04.4 live CD. The package is a complete distaster; dependencies are not installed:
#
# - on old Ubuntus, partclone was not installed
# - the compressions programs need to be installed (22.04 has zstd preinstalled)
# - smartctl is also used, although it's not strictly required
#
# In case of error, disable the GUI to see the underlying message (see https://unix.stackexchange.com/a/589650).
#
apt install -y clonezilla smartmontools
ln -s $images_parent_dir /home/partimag
```

Backup whole device:

```sh
# WATCH OUT!! The dev path is the basename!!
# If snapshotting a Windows live O/S, schedule a disk check via `chkdsk /f` (and reboot) before starting the backup.
#
# - `-j2`           : clone hidden data (between MBR and part 1; there could be e.g. bootloader code; use on disk dump)
# - `-q2`           : use partclone
# - `-z9p`          : parallel zstd (5=xz, 9=zstd, 8=lz4 (faster); `p`arallel)
# - `-p command`    : return to command prompt on completion
# - `-i 1000000`    : split size (in MB)
# - `-sfsck`        : don't run fsck
# - `-scs`          : skip check image
# - `-senc`         : skip encryption
# - `$dev_basename` : e.g. `sde`
#
# - `-c`            : ask confirmation
# - `--rescue`      : use for defective disks
#
ocs-sr -j2 -q2 -z9p -i 1000000 -sfsck -scs -senc -p command --rm-win-swap-hib savedisk $image_dir_name $dev_basename
chown -R $SUDO_USER: /home/partimag/$image_dir_name
```

Backup a single partition (discard the rest of the files):

```sh
ocs-sr     -q2 -z9p -i 1000000 -sfsck -scs -senc -p command --rm-win-swap-hib saveparts tempdir $part_dev_basename
chown $SUDO_USER: /home/partimag/tempdir/$part_image_file
mv /home/partimag/$part_image_file .
rm -rf /home/partimag/tempdir
```

Restore a whole device:

```sh
# `-g`              : grub install
# `-p`              : postaction
# `-scr`            : skip check restorable
# (`-c`             : confirm)
#
# Formerly needed: the partitions metadata files generated by the live USB version are incompatible with the Ubuntu
# version; this header is not meaningful anyway.
#
#   sed -i.bak '/^sector-size:/ s/^/# /' /home/partimag/$image_dir_name/*.sf
#
ocs-sr -g auto -e1 auto -e2 -r -j2 -scr -p command restoredisk $image_dir_name $dev_basename
```

Restore a single partition:

```sh
# MAKE SURE THAT THE DESTINATION IS THE EXPECTED ONE!!
#
# `-d`     : debug
# `-s -`   : source
# `-p ...` : output
#
# Use `dd of=/dev/sda5 bs=4M status=progress` if dd was used.
#
pv nvme1n1p4.ntfs-ptcl-img.zst.aa | zstd -d | partclone.restore -d -s - -o /dev/nvme0n1p4
```

Clone a disk:

```sh
# `-b`    : batch (no keypresses)
# `-icds` : skip check destination s8ize
# `-r`    : resize partition
#
ocs-onthefly -b -icds -r -j2 -p a -f /dev/sdc -d /dev/sda
```

## Dump CDs (safely)

DVDisaster will check the data while dumping:

```sh
dvdisaster -d $block_dev -r --read-raw -i $image.iso
```

## Sleep

Sleeping works for decimals, and multiple times!

```sh
# Sleep for 1.5 hours. Valid suffixes: `s`, `m`, `h`, `d`
#
sleep 1.5h
sleep 1h 30m
```

## Watch

Be aware that watch commands are tricky. While not quoting the command works, for more complex commands, it will fail. Additionally, the shell executed it dash.

Example of scripted invocation:

```sh
watch_command=(watch -n 1 -x) # will execute bash
output_format_options=()
query_option=("$(printf "%q" "SELECT * FROM mytable")")

"${watch_command[@]}" bash -c "mysql ${output_format_options[*]} -uroot -h$mysql_host ${schema_option[*]} ${query_option[*]}"
```

## Displaying messages

```sh
# Simple popup window with button.
# WATCH OUT! Don't forget `--text`, otherwise, a cryptic message 'All updates are complete.' is shown (!?).
#
# - `--no-markup`       : disable text interpretation ("Pango" markup)
# - `width`, `--height`
#
zenity --info --no-markup --text 'Text message!'
```

## Nohup

Avoid a process to be closed on HUP (ie. SSH disconnection):

```sh
# Can replace the stdout/err redirections with `> myprog.log 2>&1`
#
nohup myprog > myprog.out 2> myprog.err < /dev/null &
```

References:

- [general](https://stackoverflow.com/a/29172)
- [</dev/null](https://stackoverflow.com/a/19956266)

## xdotool (X11 automation)

Send key/mouse events to windows! Send Alt+F4 to a window:

```sh
# Search in all the workspaces; it's better to search by name.
# `--name` is a regex!
#
window_id=$(xdotool search --all --name smplayer | tail -n 1)
xdotool search --name 'Slack \| '

# windowactivate (bring to front) is necessary.
#
xdotool windowactivate --sync "$window_id" key "alt+F4"

# Sending to windows in the background is either not always supported, or complex (https://stackoverflow.com/a/56240192).
# The easiest thing is to store the current window, do the operations, then restore it.
xdotool getactivewindow > /tmp/activated_window
# blahblah
xdotool windowactivate "$(< /tmp/activated_window)"
```

See `browser-common.sh` for a complex example, and [StackOverflow](https://unix.stackexchange.com/q/87831) for discussion.

## Dates

General format: `date +$format --date=$expression`

Interpret a date: `date -d $input`

### Formatting

Formatting symbols (apply via `+` param):

- `%a/%A`    : short/full day of the week
- `%b/%B`    : short/full eng. month
- `%H:%M:%s`
- `%Y-%m-%d` : `%m` has leading 0
- `%F %T`    : same as `%Y-%m-%d %H:%M:S` (time without seconds: `%R`)
- `%T.%3N`   : time with milliseconds (`%N` is nanoseconds; `.%3` prints 3 digits)
- `%s`       : unix epoch
- `%-m`      : don't pad (in this case, month without leading 0)

The output language is the system one; in order to change, set `LC_ALL`:

```sh
LC_ALL=en_GB date # ...
```

Checking: `date -d $date` can be used to check if a date is valid.

### Operations

Operations via natural language expressions:

```sh
date                                # `now` is implied
date --date='yesterday'
date --date='2 years ago'
date --date='2 days'                # now plus days
date --date='next tue'
date --date='@1'                    # epoch time
```

The option `--debug` shows the logic applied.

WATCH OUT!! The `month` calculations (e.g. previous/next month) are not intuitive (see https://stackoverflow.com/a/13168625):

```sh
# Today: Di 31. Aug 12:06:26 CEST 2021

$ date --date='-2 month'
Do 1. Jul 12:06:37 CEST 2021 # not 30/Jun!

$ date --date='next month'
Fr 1. Okt 12:06:51 CEST 2021

# Workaround, if the month only is meaningful (1-based):

$ date --date="$(date +%Y-%m-15) -1 month"
Di 15. Jun 00:00:00 CEST 2021
$ date --date="$(date +%Y-%m-15) next month"
Mi 15. Sep 00:00:00 CEST 2021
```

WATCH OUT!! When performing operations with a date, the timezone must be specified, otherwise, the '+/- ...' will be interpreted as the time zone:

```sh
$ date '+%R'; date '+%R' --date '-1 hour'
22:07
21:07

# WRONG!
$ date '+%R' --date='2021/04/19 22:07 - 1 hour'
02:07

# Correct!
$ date '+%R' --date="2021/04/19 22:07 $(date +%Z) - 1 hour"
21:07
```

Arithmetic via conversion to seconds:

```sh
timestamp_secs=$(date -d "2015-12-12 13:13:28" +"%s")
result_secs=$(( timestamp_secs - 90 ))
date '+%F %R' --date="@$result_secs"

data_start_secs=$(date -d "$data_start" +"%s")
highlight_start_secs=$(date -d "$highlight_start" +"%s")
expr $highlight_start_secs - $data_start_secs
```

### Calendar

!!!

```sh
cal [[$month] $year]
```

## PDF/Images handling

PDF images extraction; WATCH OUT! Some PDFs may be complex, including "soft masks" (`smask` in pdfimages listing):

```sh
# List the images in a PDF file
#
pdfimages -list $input

# Extracts all the images (lossless) (package `poppler-utils`)
#
# -all:   save all the image types
# prefix: either the output files prefix, or, if it ends with `/`, a path (which must exist)
#
pdfimages -all $input $prefix

# Losslessly create a PDF from image files. WATCH OUT! Don't use Imagemagick (convert), as it's not lossless.
#
img2pdf -o output.pdf *.jpg
```

Other operations:

```sh
# Extract pages/remove a page from a PDF doc (lossless):
#
pdftk $input.pdf cat 1-29 31-end output $output.pdf

# Compress a pdf:
#
# Some sources use `-dPDFSETTINGS=/prepress` for the best quality, but the [manual](https://www.ghostscript.com/doc/VectorDevices.htm#PSPDF_IN)
# specifies that for the best quality, the default should be used.
# `-dPDFSETTINGS=/ebook` gives a strong compression, with still good quality.
#
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dNOPAUSE -dBATCH -sOutputFile=$out $in

# Convert an Open XML (docs) file to PDF (requires Libreoffice)
# The extension is replaced with PDF, independently of it.
# Output dir defaults to current, not the doc's!
#
lowriter --convert-to pdf $file [--outdir $dir]
```

For cropping (via GUI), best to use an online tool (e.g. [Xodo](https://pdf.online/crop-pdf)); (maintained) GUI tools don't copy bookmarks (krop, pdfarranger), or don't work as intended, or they have unintended side effects (pdfcrop increases the file size).

### Imagemagick (convert)

```sh
convert $input -flip -flop $output                # Rotate 180°
convert $input (-resize|-scale) 50% $output       # Resize (with interpolation) or scale (without) to 50%
convert -size 1920x1080 xc:white $output.pdf      # Create blank pdf page, with given resolution (size)
convert -coalesce animation.gif $output           # Split an animated gif into its frames; `-coalesce` is required for more complex sources.
convert -crop 3x3@ $input $prefix_%d.ext          # Simplest splitting of an image (3x3); see https://is.gd/cDPUxD
```

In order to resize an animated gif, use `coalesce`, then, separately, `resize`.

Preset for resizing, contrasting and compressing a scanned document:

```sh
# -resample/-units : resample to 300 DPI
# -level           : darken midtones
# -quality         : set compression quality; JPEG default is 92%
#
convert -resample 300 -units PixelsPerInch -level 0%,100%,0.5 -quality 85% "$input.ext1" "$output.ext2"

# If the above doesn't work, use:
#
convert -density 300 -level 0%,100%,0.5 -quality 85% "$input.ext1" "$output.ext2"
```

Convert a character (e.g. emoji) to a PNG file (and resize it):

```sh
# It seems it's not possible to make `pango-view` output to stdout, even via GraphicsMagick.
#
echo -e "🤙" |
  pango-view --dpi=300 --no-display --font='Droid Sans Mono' --output=call_me.png /dev/stdin &&
  convert call_me.png -resize 30x30 ../emoji_icons/call_me.png
```

### Raw to JPEG conversion

Use dcraw, Darktable or RawTherapee:

```bash
# This seems to produce nicer-looking photos than DarkTable.
#
ls -1 *.ORF | parallel 'dcraw -c {} | ppmtojpeg > {}.jpg'

# Can't be parallelized due to global lock (possibly there's a flag to disable this).
#
for f in *.ORF; do darktable-cli "$f" "$f".jpg; done
```

[Reference](https://askubuntu.com/a/1256073/46091).

## Formatting/diff operations

```sh
# Table (columns) formatting
#
# `-t`: automatically determine columns based on whitespace
#
# For tabs, use $'\t'
#
# In order to format a diff, normalize the inputs, and reformat the diff output:
#
#   diff <(tr $'\t' ' ' <<< "$reference") <(tr $'\t' ' ' <<< "$current") | column -s $'\t' -t
#
#
column -s $separators -t

# Diff (subtract) lines of two files (file_2 - file_1).
# The two files don't need to be sorted.
# When using in a script, append `|| true`, otherwise it will fail if the result is empty.
#
grep -Fvx -f $file_1 $file_2

# Find common lines between two files; must be both sorted (lexicographycally)
#
comm -12 $file_1_sorted $file_2_sorted
```

## Encoding

```sh
file -i $filename                        detect file encoding
enca -L none $filename                   ^^
iconv -f UCS-2 -t UTF-8 $filename        converts file encoding

# convert binary input to something human readable (eg. hexadecimal)
#
hexdump -C
# `-e`: custom format; every byte (`/1`) the lowercase hex value), and every 32 bytes (`/32`) the newline (NL is not printed by default)
#
hexdump -e '/1 "%02x"' -e '/32 "\n"'

# convert binary to base64, and back [hexadecimal,human readable].
#
base64 [-i <input>] [-o <output>]
base64 -d [-i <input>] [-o <output>]
```

## Benchmarking

The `time` program is built-in in modern shells. For more userful functionality, use `/usr/bin/time`.

```sh
# `-f`/`--format`: better use `-f`, which is more portable.
#
# %e: walltime
#
# The output goes to stderr!
#
/usr/bin/time -f ">>> WALLTIME:%e"
```

If millisecond precision is required, use the [Bash built-in](bash.md#time-related-functionalities-incl-benchmarking).

## Images

### Mounting images

Mount raw dumps:

```sh
# Attach the image to a loop device (**this is not mounting**)
#
loop_device=$(sudo losetup --show --find --partscan $image)

# List attached devices
#
losetup -l
#
# NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE                                               DIO LOG-SEC
# /dev/loop0         0      0         0  0 /home/saverio/code/riscv_images/components/busybear.bin   0     512

# Now we can mount
#
sudo mount $loop_device /mnt

# Detach the loop device
#
sudo losetup -d $loop_device
```

Mount any image type, via `qemu-utils`:

```sh
# Use `-f vpc` for vhd images.
#
qemu-nbd --connect=/dev/nbd0 [-f vpc] $image

# Inform the kernel of the changes.
#
partprobe /dev/nbd0

# List the partitions (optional); doesn't need any wait (only partprobe).
# Can also read a raw image!
#
fdisk -l /dev/nbd0

# Complete clusterduck. Even waiting via:
#
#     while [[ ! -b /dev/nbd0p4 ]]; do sleep 0.1; done
#
# will cause:
#
#     ls -l /dev/nbd0p4
#     ls: cannot access '/dev/nbd0p4': No such file or directory
#
# Therefore, we use `ls` instead.
#
while ! ls -l $partition_device > /dev/null 2>&1; do sleep 0.1 done

mount "$partition_device" /mnt
```

### Create an iso from a directory

```sh
# Use [J]oliet FS and [R]ock Ridge, for better compatibility with filesystem features
#
mkisofs -JR -o $outfile.iso $dir
```

## Create patches (diff)/restore them

```sh
# Create the diff (`u`nified format)
#
diff -u $original $modified > patch.diff

# Create the diff from git (see git notes for details)
#
git diff --no-prefix $file > patch.diff

# `p0`: required for files in subdirs; for safety, best to always use it.
#
patch -p0 $patch.diff
```

If a hunk has been applied, patch will prompt. It's possible to skip this by using `--forward`, although that will return `1` exit status.

## Remote desktop

Remove desktop on a headless server.

Broken (disconnects) - just use Nomachine; kept here for reference.

References:

- https://stackoverflow.com/a/40678605
- https://www.richud.com/wiki/Ubuntu_Fluxbox_GUI_with_x11vnc_and_Xvfb

Server configuration

```sh
 fluxbox: simple diplay manager

sudo apt install x11vnc xvfb fluxbox

# Short way
#
# - `-no6 -noipv6 -rfbportv6 -1`: for systems with ipv6 disabled (see https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=672449)
# - `-forever`                  : without it, the service will terminate after the first session
# - `-ncache`                   : suggested, but corrupts the screen

x11vnc -create -env FD_PROG=/usr/bin/fluxbox \
  -env X11VNC_FINDDISPLAY_ALWAYS_FAILS=1 \
  -env X11VNC_CREATE_GEOM=${1:-1280x800x16} \
  -gone 'killall Xvfb' \
  -nopw \
  -forever \
  -bg
  -no6 -noipv6 -rfbportv6 -1
```

Client:

```sh
sudo apt install tigervnc-viewer

# `-N`                              : no command; just proxy
# `-T`                              : disable pseudo-terminal allocation
# `-L $local_socket:$remote_socket` : proxy!
#
ssh -N -T -L 5900:localhost:5900 arcade &

xtigervncviewer localhost:5900
```

## Docker

```sh
# Stop all containers
docker stop $(docker ps -a -q)

# Remove all containers
docker rm $(docker ps -a -q)

# Remove all images
docker rmi $(docker images -a -q)
```

## Wine

Install .NET:

```sh
# It's not clear if `--force` is required.
#
winetricks dotnet472 corefonts
```

## Build the Linux kernel

Explanation of the Ubuntu kernel versioning [here](https://ubuntu.com/kernel).

Repositories:

- `git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git`: reference
- `git@github.com:torvalds/linux.git`: alternative, lags a bit
- `git://git.launchpad.net/~ubuntu-kernel/ubuntu/+source/linux/+git/focal`: Canonical versions for the given release, including HWE etc. (see [here](https://wiki.ubuntu.com/Kernel/Dev/KernelGitGuide))

Run `make help` to display the targets help.

Procedure:

```sh
# (Packages installation is not included)

# Copies the running kernel config (if .config is not present), and apply defaults for the new settings
# of the new kernel version.
# The running kernel config can be found in `/boot`.
#
make olddefconfig

# Necessary on some Debian/Ubuntu configs.
#
# References:
# - https://cs4118.github.io/dev-guides/debian-kernel-compilation.html
# - https://askubuntu.com/q/1329538
#
scripts/config --set-str SYSTEM_TRUSTED_KEYS ""
scripts/config --set-str SYSTEM_REVOCATION_KEYS ""

# Without these, a large debug info package is generated.
#
# From GUI: "Kernel hacking" -> "Compile-time checks and compiler options" -> "Compile the kernel with debug info"
#
# Programmatically (less safe, since in theory, the settings may change):
#
scripts/config --undefine DEBUG_INFO
scripts/config --undefine DEBUG_INFO_COMPRESSED
scripts/config --undefine DEBUG_INFO_REDUCED
scripts/config --undefine DEBUG_INFO_SPLIT
scripts/config --undefine GDB_SCRIPTS
scripts/config --set-val DEBUG_INFO_DWARF5 n
scripts/config --set-val DEBUG_INFO_NONE y

# Example of disabling entries.
#
scripts/config --disable CONFIG_CPU_SUP_HYGON

# There are two GUI options for interactive configuration.
# The easiest way to turn interactive changes into non-interactive is to use the interactive mode,
# then compare the old and new version via `scripts/diffconfig`.
#
# Remember that there are different `scripts/config` commands:
#
# `--undefine`: entirely remove
# `--disable`:  comment
# `--enable`:   uncommment
# `--set-val`:  set a value
# `--set-str`:  set a quoted value
#
make xconfig    # X11
make menuconfig # terminal
```

Build:

```sh
# This also runs `make clean`, and builds the kernel (binary packages only; no source ones).
# If there are errors, the last error message is not informative; either scroll up, or run without
# `-j` (which makes the last error message informative).
# All the deb packages are required.
#
# The firmware files are not included, so include the latest `linux-firmware` package, if needed.
# The official firmware repo is https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git.
#
make -j $(nproc) bindeb-pkg LOCALVERSION=-sav

# If the build is interrupted, run these, otherwise wacky errors may happen, and restart from scratch.
# mrproper does not checkout (git) changed files.
#
make mrproper
rm -rf ../$(basename "$(pwd)").orig
```

Old references:

- [What's a simple way to recompile the kernel?](https://askubuntu.com/questions/163298/whats-a-simple-way-to-recompile-the-kernel)
- [BuildYourOwnKernel](https://wiki.ubuntu.com/Kernel/BuildYourOwnKernel)
- [Compiling the kernel with default configurations](https://unix.stackexchange.com/questions/29439/compiling-the-kernel-with-default-configurations)
- [What does “make oldconfig” do exactly in the Linux kernel makefile?](https://stackoverflow.com/questions/4178526/what-does-make-oldconfig-do-exactly-in-the-linux-kernel-makefile)
- [Where can I get the 11.04 kernel .config file?](https://askubuntu.com/questions/28047/where-can-i-get-the-11-04-kernel-config-file)
- [make config vs oldconfig vs defconfig vs menuconfig vs savedefconfig](http://embeddedguruji.blogspot.com/2019/01/make-config-vs-oldconfig-vs-defconfig.html)

### Building the Ubuntu mainline kernel

The mainline kernel is *not* built the standard way; the .config file is not directly involved.

A good reference is the [`tuxbuilder` builder](https://github.com/TuxInvader/focal-mainline-builder/blob/main/build.sh), which does, more or less:

```sh
git clone --depth=1 -b cod/mainline/v6.0.3 git://git.launchpad.net/~ubuntu-kernel-test/ubuntu/+source/linux/+git/mainline-crack
cd mainline-crack

# Must build only for amd64 (see [issue](https://github.com/TuxInvader/focal-mainline-builder/issues/30)):
#
perl -i -pe 's/archs="amd64\K.+/"/' debian.master/etc/kernelconfig

# Specific GCC versions are required for each Canonical kernel build; when not installed, the error
# is not obvious (`WARNING: x86_64-linux-gnu-gcc not installed`).
# This makes sure that the correct GCC version is installed.
#
apt install -y "$(perl -ne 'print /gcc\?=(gcc-\d+)/' debian/rules.d/0-common-vars.mk)"

fakeroot debian/rules clean defaultconfigs
fakeroot debian/rules clean

# Use `build=source` to build the source package, although it requires extra configuration.
#
# Above, the archs other than amd64 have been removed; without that step, this command requires `-aamd64`.
#
dpkg-buildpackage --build=binary -d

# The metapackage requires extra configuration; see `do_metapackage()`.
```

#### Mainline configuration

The mainline configuration is long and complex; it seems that it's built from a script outside the repo. A log is [here](https://kernel.ubuntu.com/~kernel-ppa/mainline/v6.1/amd64/log).

The Canonical configurations are under `debian.master/config/amd64` (only `cod/` branches have `debian.master`); they're not included in the `defconfig` target, and can be merged, but they won't build with the standard method. Running `make olddefconfig` alone from a `cod/` branch generates the vanilla config.

Versions from 6.1 onwards include additional packages; one of them is `linux-buildinfo`, which includes the configuration, but it seems that it's not possible to build that alone.

### Other config notes

It's not straightforward/possible to extract the kernel config from non-running kernels, for example, the script `extract-ikconfig` requires a certain compile-time setting.

A workaround is to install the Ubuntu modules package, which will install the config under `/boot` (after copy, the package can be uninstalled).

If it's not found how to generate the Ubuntu mainline config from the mainline-crack repo, in theory one could set `CONFIG_IKCONFIG=m`, build the kernel, extract the image, and run `extract-ikconfig`.

When performing a standard build of the deb packages, the package `linux-image` contains a copy of the configuration; the Ubuntu package doesn't, though.

Configurations can be merged via script: `scripts/kconfig/merge_config.sh .config debian.master/config/amd64/config.*`.

The frame size errors can be ignored by setting `CONFIG_FRAME_WARN=0` (but this should not be necessary in first place).

The arch config can be found at `https://raw.githubusercontent.com/archlinux/svntogit-packages/packages/linux/trunk/config`, but it's very different from the Ubuntu one.
