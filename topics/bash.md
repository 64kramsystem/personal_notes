# Bash

- [Bash](#bash)
  - [Shell key bindings](#shell-key-bindings)
  - [Shellopts](#shellopts)
    - [Related operations](#related-operations)
    - [Madness references](#madness-references)
  - [Special variables](#special-variables)
  - [Variables](#variables)
  - [Strings](#strings)
  - [Herestrings/Heredocs (+stdin handling)](#herestringsheredocs-stdin-handling)
  - [Special functions](#special-functions)
  - [Flow control](#flow-control)
  - [Test conditions](#test-conditions)
    - [Regular expressions](#regular-expressions)
    - [`expr` tool (also for regular expressions)](#expr-tool-also-for-regular-expressions)
    - [Applications](#applications)
  - [Functions](#functions)
  - [Grouping commands](#grouping-commands)
  - [Background processes/jobs management](#background-processesjobs-management)
  - [String functions](#string-functions)
    - [Examples](#examples)
  - [Paths](#paths)
  - [printf (escaping/formatting)](#printf-escapingformatting)
  - [Arithmetic operations](#arithmetic-operations)
    - [Random values](#random-values)
  - [Redirections](#redirections)
  - [Substitutions](#substitutions)
  - [Pipes error handling](#pipes-error-handling)
  - [Wildcards/ranges](#wildcardsranges)
  - [Arrays](#arrays)
    - [Snippets](#snippets)
  - [Associative arrays](#associative-arrays)
  - [Shellcheck](#shellcheck)
  - [Script operational concepts/Useful scripts](#script-operational-conceptsuseful-scripts)
    - [Split a string (single/multi-char delimiter) into an array](#split-a-string-singlemulti-char-delimiter-into-an-array)
    - [Ask for input (keypress)](#ask-for-input-keypress)
    - [Error handling](#error-handling)
      - [Simulating error bubbling (unexpected `set -o errexit` behavior)](#simulating-error-bubbling-unexpected-set--o-errexit-behavior)
      - [Trapping errors (hooks)](#trapping-errors-hooks)
    - [Log a script output/Enable debugging (log)](#log-a-script-outputenable-debugging-log)
    - [Print to stdout while setting a variable](#print-to-stdout-while-setting-a-variable)
    - [Check if there's data in stdin](#check-if-theres-data-in-stdin)
    - [Check if a script is `source`d](#check-if-a-script-is-sourced)
    - [Sudo-related tasks](#sudo-related-tasks)
    - [Time-related functionalities (incl. benchmarking)](#time-related-functionalities-incl-benchmarking)
    - [Parse commandline options (`getopt`)](#parse-commandline-options-getopt)
    - [Poor man's configfile parser](#poor-mans-configfile-parser)
    - [`bash` command options](#bash-command-options)
    - [Variables printing function](#variables-printing-function)
    - [Current shell/OS](#current-shellos)
    - [Convert associative array to JSON](#convert-associative-array-to-json)
  - [Pitfalls](#pitfalls)
  - [Shell colors](#shell-colors)

## Shell key bindings

- `Ctrl+U` delete line before cursor  `Ctrl+K`  delete line after cursor
- `Ctrl+A` go to BOL                  `Ctrl+E`  go to EOL
- `Ctrl+W` delete word before cursor  `Alt+D`   delete word after cursor
- `Ctrl+L` clear

## Shellopts

Basic (important):

```sh
# Shell opts can't be set on a single line

set -o pipefail
set -o errexit
set -o nounset
set -o errtrace
shopt -s inherit_errexit
```

Notes:

- `nounset`         : WATCH OUT! Unset arrays don't cause errors
- `errtrace`        : (`-E`) trap errors also inside functions
- `inherit_errexit` : subshells inherit errexit (Bash 4.4+). WATCH OUT!! Doesn't always work (see [madness](#madness-references))

Extra:

```sh
set -o monitor            # (`-m`) enable job control
shopt -s nullglob         # IMPORTANT: when globs don't match anything, expand to null string, rather than to themselves
set -o xtrace             # (`-x`) debugging mode; prints all the statements
shopt -s nocasematch      # case insensitive matches
```

If the xtrace is set before a backgrounded block (e.g. `{ while ... } &`), it won't be logged.

See http://linuxcommand.org/lc3_man_pages/seth.html.

### Related operations

```sh
set +o shellopt                           # disables a shellopt
[[ $SHELLOPTS =~ $(echo '\bnounset\b') ]] # clean/easy way to test shellopts
export SHELLOPTS                          # pass the shell options to subshells! WATCH OUT: recursive
env SHELLOPTS=$SHELLOPTS mycmd            # ^^ (same)
```

### Madness references

- https://mywiki.wooledge.org/BashFAQ/105

The `inherit_errexit` shopt doesn't work with process substitution:

```sh
set -o errexit; shopt -s inherit_errexit

mapfile -td. latest_version < <(false) # !!! SUCCEEDS !!!

# Use this instead:
#
command_output=$(...)
mafile -td. latest_version <<< "$command_output"
```

## Special variables

- `$?`       exit code of last executed process; must test in a separate (via `;`) command
- `!!`       last command
- `!$`       last argument of last command
- `!<n>`     executes the `<n>`th command
- `$$`       current pid
- `$!`       pid of last backgrounded process; doesn't work with `sudo -b`, which requires manual processing
- `IGNOREEOF=$n` prevents Ctrl+D from exiting (also from sudo) immediately, requiring doing it ($n + 1) times.

Parameter variables; WATCH OUT!! When inside a function, they refer to the function:

- `$<n>`     parameters passed; n in [0...9], 0 = command; also valid in alias!
- `$@`       or $* all parameters; preferrable to use the $@, which handles spaced params
- `$#`       number of params, excluding #0

## Variables

```sh
myvar=value                         # no spaces around equal; if value includes spaces, it must be quoted
myvar=$myvar2                       # $myvar2 doesn't require quotes
unset myvar                         # delete myvar
export myvar=value myvar2           # makes available to subshells. `local export` is allowed, but doesn't work as intended; in order to export
                                    # local variables, do it separately from declaration, or declare via `declare -x`
local myvar1 myvar2                 # declare multiple local variables in one statement
((var+=1))                          # increment variable value
((var++)) || true                   # increment variable value. WATCH OUT!!! `|| true` is required if var=0 before the increment/decrement!!!!

# Indirect variable referencing; works also with associative arrays!!
#
declare -n var_ref=var              # bash only; `-n` creates a reference; WATCH OUT! `unset` will unset also the source var!
eval local var_ref=\$$var           # works both on bash/zsh; this shows how to use local modifier

myvar=$(nonexiting_command 2>&1)    # the assignment value is stdout's output; for stderr we need redirection (eg. see whiptail)

declare -g                          # declare variable as global (eg. from a function); without, the scope is the current one (local)
declare -x                          # export; can add to `-g`
```

## Strings

```sh
# Interpret new lines!. It won't be interpolated when inside double quotes.
#
echo $'l\'s\n'          # prints `l's` and newline

# Best way to escape the exclamation mark inside double quotes.
# WATCH OUT! Backslash doesn't work.
#
"foo"'!'"bar"

# If the variable is a string, can slice using a negative value for the end index (!!)
#
slice=${str[*]:1:-1}

# Interpolate strings in string quotes. Requires no double quotes to be in $str.
#
# Working:
#
# - the inner double quotes are for correctly quoting $str;
# - the outer double quotes are for not breaking str into multiple lines; the first line would be interpreted
#   as echo, but the following as other commands.
#
# This logic doesn't allow double quotes, because it's not possible to distringuish which ones to escape,
# if there are at multiple scopes (e.g. outside and inside a command substitution).
#
eval echo \""$str"\"
```

## Herestrings/Heredocs (+stdin handling)

Herestring: Bash alternative for "echo X | command", with the difference that it doesn't create a subshell.
WATCH OUT: it adds a newline, which in some cases is undesirable.

```sh
command <<< X
```

Heredoc:

```sh
# The heredoc data is sent to stdin - it's not a parameter!
#
ruby -ne 'puts $_' <<STR
$this_is_interpolated
STR

ruby -ne 'puts $_' <<'STR'
$this_is_not_interpolated
STR

# Indentation. WATCH OUT! The prefix is tabs, not spaces!!
#
  ruby -ne 'puts $_' <<-STR
    can be indented
  STR # <-- prefix this with tabs
```

Assignment/passing:

```sh
# Check if stdin/heredoc is passed.
# WATCH OUT this is timing-based, so it won't work for anything that sends data with a delay.
#
if read -t 0; then ...; fi

# Assign stdin to a variable
#
read -r myvar           # single line (WATCH OUT!!); `-r`: don't interpret backslashes
myvar=$(</dev/stdin)    # multi-line, but will block if there is no input!
myvar=$(cat)            # (same)

# Assign a variable via heredocs.
# See https://stackoverflow.com/q/1167746 for comments.
myvar="$(cat << JSON
{
  "title": "$1",
  "body": "$2",
}
JSON
)
```

## Special functions

```sh
# Brace expansion:
#
cp file{,.bak}

# !!! Sequence (range) !!!
# !! Works also with leading 0s !!!
#
for i in {003..005}; do echo $i; done    # prints "003 004 005"

# Expand a files content into parameters. !!! Don't use if escaping is required (e.g. spaces) !!!
# This works because redirection without an input process (`< $filename`) prints the file content.
#
rubocop --auto-gen-config "$(< "/tmp/files_list.txt")"
```

## Flow control

```sh
# If/else
#
if $condition; then
  $commands
elif $condition; then                   # don't forget to follow with `then` !!
  $command
else
  :                                     # denotes an empty block
fi

# Switch/case. Double semicolon is required (can be put at the end of a statement); space before `)` isn't.
#
case $env_number in
*[0-9][0-9][0-9]*r )
  foo
  ;;
[yY] | [yY][Ee][Ss] )
  bar
  ;;
[nN] | [n|N][O|o] )
  baz
  ;;
*)
  boom
  ;;
esac

# For loop
#
for s in host1 host2; do
  echo "$s"
  continue                    # skip current iteration
  break                       # break the loop
done

# !!! C-style for loop !!!
#
for ((job = 1, othervar = 0; job <= MAX_JOBS; job++)); do $command; done

# Implied "$@"
#
for val; do echo "$val"; done

# Until loop
#
until (( $VGAPT_MEMORY < $(free -m | sed '2q;d' | awk '{print $NF}') )); do $command; done
```

## Test conditions

```sh
test $expr                              # test an expression

if [[ ( $expr1 ) || ( $expr2 ) ]]       # complex conditionals; parenthesis denote priority
if { $commands; } || { $commands; }     # conditional with commands (for simple commands, braces+semicolons are not required)
if (( lhs < rhs ))                      # arithmetic comparison (dollars are not required)
```

WATCH OUT!! In "compact conditional statements", the conditional itself doesn't cause an exit (because it's a conditional), but dependent commands can:

```sh
set -o errexit
[[ $var == foo ]] && true                    # never cause an exit
[[ $var == foo ]] && echo "$bar" | grep baz  # can exit, due to grep !!!
```

Note that the left side of a condition (or the only one on prefix operators) doesn't need to be escaped.

Prefix Operators:

- `!`                   not

- `-z $string`          string is empty
- `-n $string`          string is not empty

- `-v var_no_dollar`    for scalars: test if variable has been declared/defined; WATCH OUT: don't prefix with the dollar!
                        interpolation is allowed (`-v $indirect_var`), to test a var whose name is stored in another

- `-e $filename`        file/directory/symlink exists; symlink is followed before checking!
- `-f $filename`        file exists
- `-x $filename`        file exists and it's executable
- `-d $dirname`         directory exists
- `-b $dev`             block device exists
- `-L $filename`        file/directory is a symlink
- `-b $filename`        file is a block device

- `-s $filename`        file exists and it has any char in it; negate to test empty/nonexisting file

- `-t 0`                input is terminal (false if stdin)

Infix operators:

- `-a`, `-o`                           boolean and/or
- `$string == *"substring"*`           substring matching; strings can have spaces; use this exact syntax!
- `-eq`,`-ne`,`-lt`,`-le`,`-ge`,`-gt`  arithmetic operators; !! DON'T USE FOR STRINGS !!
- `==`, `!=`, `<`, `>`                 string operators - !! DON'T USE FOR ARITHMETIC !!

Ternary operator:

```sh
# It exists, but it works only in an arithmetic context (can't use strings)
#
tern=$(( a < b ? c : d ))

# A compact conditional can be used otherwise (make sure the commands evaluate to true!)
#
tern=$( [[ -n $v_smt_on ]] && echo 2 || echo 1 )
```

Note that functions can return a condition result!!:

```sh
function myTest { [[ "$1" == "foo" ]]; }
myTest "foo" && echo "bar"      # prints `bar`!
```

### Regular expressions

For non-trivial expressions, better use `grep -qP`!

!! WATCH OUT !!

1. They're not PCRE;
2. Don't quote the regex (use backslahes to escape spaces, or `$(echo -n "...")`)!!
3. Use a temporary variable (or cmd subsitution) when using metacharacters with backslash (but not `.`);
4. `^` and `$` refer to the *whole string*, not the line;
   - the pattern represents a substring
5. DON'T WRAP with ruby regex slashes!!!

Metacharacters:

- `* + ?`      : supported
- `\w`         : not supported; use `[:alnum:]`, but *doesn't match underscore*
- `\d`         : not supported; use `[0-9]` or `[:digit:]`
- `[:alpha:]`  : supported
- `\S`         : not supported
- `\b`         : not supported; use `[:<:]` and `[:>:]`
- `\s`         : `[:space:]`
- `[:xdigit:]` : hexadecimal!
- `[^…]`       : supported
- `{}`         : supported, also for intervals (also with empty limits)
- `()`, `|`    : supported
- `(?:…)`      : not supported

References:

- Overview: http://tldp.org/LDP/abs/html/x17129.html
- Chacter classes: https://www.gnu.org/software/grep/manual/html_node/Character-Classes-and-Bracket-Expressions.html
- Limitations: https://stackoverflow.com/a/12696899

The captured groups are available in the `$BASH_REMATCH` array (with entry 0 being the full match).

Examples:

```sh
[[ 192.168.1.2 =~ ^[0-9]{1,3}(\.[0-9]{1,3}){3}$ ]] && echo MATCHES
re='\bword\b'; [[ ' word ' =~ $re ]] && echo MATCHES
[[ ' word ' =~ $(echo '\bword\b') ]] && echo MATCHES
```

### `expr` tool (also for regular expressions)

Substring extraction:

```sh
# Via regexp:
#
# - the expression must match from the beginning
# - the expression must be in a group
# - escaping is not clear; parentheses must be escaped
#
expr match "ab1b2" '.*\(b.\)'     # `b2`
expr match "ab1b2" '.\*\?\(b.\)'  # `b1`

# Via position:
#
expr substr $input $position_1_based $length
```

### Applications

Most standard way to test if a binary/executable is in PATH/exists (dash-compatible):

```sh
# Dash
if ! [ -x $(command -v $binary) ]; then error...

# Bash
! command -v $binary > /dev/null
```

Execute a command if a process is not running:

```sh
! pgrep -fa mysqld && (mysqld &)                       # Zsh doesn't need brackets for this semantics
if [[ -z "$(pgrep -fa mysqld)" ]]; then mysqld & fi
```

## Functions

```sh
# When writing a function on a single line, the command must end with a semicolon (';')
#
function myfx() {
  # Params are passed as $n (1-based)
  #
  echo "${1}"

  # WATCH OUT! `declare` inside a function is local! must use `-g` if not.
  #
  declare -g myvar=123

  # Print the function names in the stack (0-based)
  #
  echo ${FUNCNAME[0]} # `myfx`

  # Pseudo-return value (!!!)
  #
  { myvalue=$(myfunction 3>&1 1>&4); } 4>&1
}
```

## Grouping commands

```sh
# Notice the difference between the `;` requirement
#
( commands  )  # creates a subshell
{ commands; }  # runs in current context
```

The exit status is the exit status of `commands`. It's not clear why `{}` creates a new process, while `()` doesn't (intuitively, it should be the opposite).

WATCH OUT! Killing a (sudoed) background command by PID is an unholy mess, for a few reasons:

1. kill the parent of a process doesn't necessarily kill the children
2. `{}` creates a process, while `()` doesn't
3. `sudo command` crates two processes, a `sudo` one, and a child `command` one.

Using the `()` commands group:

```sh
( sudo perf sleep 6000 ) &

# $! is 960068
#
#  960066 pts/6    S+     0:00  |   \_ /bin/bash /tmp/mytest.sh
#  960068 pts/6    S+     0:00  |       \_ sudo perf record sleep 6000
#  960070 pts/6    Sl+    0:00  |       |   \_ /usr/lib/linux-tools/5.8.0-49-generic/perf record sleep 6000
#  960073 pts/6    S+     0:00  |       |       \_ sleep 6000

# `kill`ing the sudo won't work; `pkill -P` is required.
# Using process groups requires preparation via `setsid`.
#
pkill -P $!
```

Using the `{}` commands group:

```sh
{ sudo perf sleep 6000; } &

# $! is 966104
#
#  966102 pts/6    S+     0:00  |   \_ /bin/bash /tmp/mytest.sh
#  966104 pts/6    S+     0:00  |       \_ /bin/bash /tmp/mytest.sh
#  966106 pts/6    S+     0:00  |       |   \_ sudo perf record sleep 6000
#  966107 pts/6    Sl+    0:00  |       |       \_ /usr/lib/linux-tools/5.8.0-49-generic/perf record sleep 6000
#  966110 pts/6    S+     0:00  |       |           \_ sleep 6000

# Nothing works here!
```

Working around with a subshell and a pidfile; this is likely the best:

```sh
sudo sh -c 'echo $$ > /tmp/cmdgroup.pid; perf sleep 6000;' &

# Pidfile content: 976668
#
#  976664 pts/6    S+     0:00  |   \_ /bin/bash /tmp/mytest.sh
#  976666 pts/6    S+     0:00  |       \_ sudo sh -c echo $$ > /tmp/perfshell.pid; perf record sleep 6000
#  976668 pts/6    S+     0:00  |       |   \_ sh -c echo $$ > /tmp/perfshell.pid; perf record sleep 6000
#  976669 pts/6    Sl+    0:00  |       |       \_ /usr/lib/linux-tools/5.8.0-49-generic/perf record sleep 6000
#  976672 pts/6    S+     0:00  |       |           \_ sleep 6000

# Most solid - it will send the signal to the intended process.
#
sudo pkill -P $(< /tmp/cmdgroup.pid)
```

## Background processes/jobs management

Jobs:

```sh
jobs                # job currently running; use tipically when `there are stopped jobs`
kill %[-]<n>        # kill nth (1-based) job; if `-`, start from the end (-1 = last)
kill $!             # kill latest backgrounded job
wait $pid           # wait for the process to finish; if no $pid is provided, all the baground jobs are waited

(echo abc &)        # avoid printing the job number
```

Case with piped command:

```sh
sleep 100 | tee /dev/null &
echo $!                       # returns tee's pid!
jobs -p | tail -n 1           # pid of the latest backgrounded job (`-p`= pids of group leaders)
```

In order to get the exit status of a backgrounded process, use `wait $pid`, and inspect its exit status (`$?`)``:

```sh
$ false &
[1] 790772
$ wait 790772
[1]+  Exit 1                  false
$ echo $?
1

$ true &
[1] 791005
$ wait 791005
[1]+  Done                    true
$ echo $?
0
```

Background jobs + command substitution blocking:

```sh
# Must disconnect from stdout in order not to block.
#
myvar=$(sleep 5             & echo b)  # blocks
myvar=$(sleep 5 > /dev/null & echo b)  # doesn't block

# Same; without `> /dev/null`, it blocks.
#
function bkg_function {
  { sleep 60; } > /dev/null &
  echo -n result
}
result=$(bkg_function)
```

## String functions

Substitutions generally have the single (single operator) or global (double operator) version; where this is the case, the global version is presented.

Parameter expansion supports internal expansion, and escape chars: `line=${line//$'\t'/ }`

WATCH OUT! Replacements are greedy. Also, after a replacement if performed, Bash starts from the next unreplaced substring.

```sh
${str:-<expr>}                    # default a parameter: set if undefined or blank
${str:+<expr>}                    # if the var is set, replace with <expr> (which can include the $param itself!)

${str//search/replace}            # substitution (!! NOT REGEX !!) ->
${str//[a-e]/ }                   # -> but supports at least character classes!
                                  # WATCH OUT! In <search>, `~` must be escaped (not not in <replace>).

${str:[<start>][:<end_expr>]}     # substring (0-based)
                                  # `start`: inclusive; blank: first char
                                  # `end_expr`: not inclusive; positive: length; negative: !! -1 = beforelast !!; blank: until end

${str^^}                          # uppercase (single `^` applies only once)
${str,,}                          # downcase (single `,` applies only once)
```

(g)sub (applied to filenames):

```sh
${filename%<.ext>}                # strip specific <ext>ension; `*` can be used [replace suffix]
${filename%%.*}                   # strip any extension (anything after the first dot)
${filename##*.}                   # extract (last) extension, without the dot [replace prefix]
${filename##*/}                   # extract basename, eg. `"${PWD##*/}"`

${filename%/*}                    # parent dir of a file; DON'T USE WITH DIRS! it doesn't distinguish trailing slashes.

str=aaa_foo_bbb_foo_ccc
${str%foo*}bar${str##*foo}        # replace last occurrence; WATCH OUT! The search string must be present
```

Fun tricks/other stuff:

```sh
printf '<SAV>%.0s' {1..10}            # repeat a string

... | xargs                           # strip/trim leading and trailing whitespace; compresses spaces; !! adds a newline !!
... | tr -s ' '                       # squeeze (compress) consecutive sequences of the given char
... | perl -pe chomp                  # strip one trailing whitespace,
... | perl -0777 -pe 's/^\s+|\s+$//g' # trim leading and trailing whitespace

printf "%05d\n" $n                    # add leading zeros; this is a standard printf, so the standard functionalities apply
```

### Examples

Substrings (0-based; -1=last):

```sh
${str:1}                          # chars from i=1
${str: -3}                        # last three chars; WATCH OUT! the space is required
${str::-1}                        # chars until the last (not included)
```

## Paths

Unix tools (not bash functions):

```sh
realpath $filename  # File.expand_path
basename $filename  # extract basename!
dirname $filename   # parent dir name. works with dirs, but *must* expand path if filename equals `.`/`..`
```

## printf (escaping/formatting)

```sh
printf "%q" "$str"                # escape a string
printf %02d "$i"                  # pad number with zeros
```

## Arithmetic operations

Watch out!!:

- bash arithmetic handles only integers (use `bc` for floating point)
- use brackets!!! priority is not as expected: `2 & 1 == 0` == `2 & (1 == 0)` == `0` (false)
- empty string equals to 0
- operations *can* cause exit if they evaluate to 0 and the `errexit` shellopt is set (also applies to `let`)
- command substitutions inside arithmetic expansion will not exit if they fail

```sh
(( $expr ))                       # preferred form
let param=$expr                   # alternative

b=$((++a))                        # variables don't need `$`
((a += 1))                        # `+=` for numbers
((a *= 2))                        # supported
((a ** 2))                        # power
((a || b && c))                   # boolean operators supported
((a & b | c))                     # bitwise operators supported
let '! a'; ((! a))                # negation (quotes are necessary for let; space is necessary for brackets)

`true`/`false`                    # not supported in arithmetic...
((! a))                           # ... however, 0/'' are evaluated as false, and any other number as true (strings
                                  # will fail)

a+=1                              # WRONG!!! this is a string operation (if not an arithmetic expression)

# Exits with error status:

(( expr ))                        # exits if `expr` is 0
(( val++ ))                       # exits if the old `val` value is 0
(( ++val ))                       # doesn't exit!
```

### Random values

```sh
# [0, 32767]
#
echo $RANDOM

# From `coreutils` package; Bash doesn't provide any easy way
# Limits are inclusive; `-n $n`: select $n numbers; each value ends with a newline.
#
shuf -i 2000-65000 -n 1
```

## Redirections

```sh
&> "$filename"                                          # equivalent to `1> "$filename" 2>&1`
>&2 echo "error"                                        # write to stderr

{ mycommand 3>&2 2>&1 1>&3 | grep -v "skipme" >&3; } 3>&2 2>&1  # apply a filter to stderr output (see https://stackoverflow.com/a/40336333)

exec 200> "$filename"                                   # associate a file to a file descriptor (create if not existing)

# Send stdout/err to two different outputs! See https://stackoverflow.com/a/692407.
#
my_command > >(tee -a stdout.log) 2> >(tee -a stderr.log >&2)
```

## Substitutions

Process substitution: creates a file (descriptor) from the command.

WATCH OUT!! Failing subtituted commands don't cause P.S. to exit with error!! See https://unix.stackexchange.com/q/376114.

For input file; typically used to feed output of multiple commands to another command:

```sh
meld <(command_list_1) <(command_list_2)
```

For output file; typically used with tee, to both print to stdout and redirect to a pipeline:

```sh
# If we do `| tee | xsel -ib`, the output is not printed!
#
echo "abc" >(xsel -ib)
```

Command substitution:

```sh
echo "$(command)"

# Syntax to replace a filename with the file content ([reference](https://wiki.bash-hackers.org/syntax/expansion/cmdsubst)):
#
echo $(< "$filename")

# The substitution syntax is actually optional.
#
< "$filename" command
command < "$filename"
< "$filenameL" command < "$filenameR" # both are printed, L -> R
```

WATCH OUT! See the [background processes section](#background-processesjobs-management) for notes about cmd substitution + background jobs.

## Pipes error handling

Pipes in bash have a very serious issue - in case of write error, a reader only gets an EOF, not an error.

In order to work this around, there are two approaches.

If it's acceptable for readers cleanup to be performed separately, inspect `PIPESTATUS[@]` (one entry for each pipe command) after the failure and act accordingly.

If it isn't, and readers must be killed before receiving EOF, and a complex (and ugly) approach is required - see [here](https://stackoverflow.com/a/32699218/210029)

## Wildcards/ranges

```sh
# Brace expansion: generate a sequence; $end is included. Spaces and expressions are not allowed; better
# use arithmetic for loop.
# WATCH OUT! If used as files prefix (`{a..z}*`), also non-matching entries are included.
{$start..$end[..$incr]}

# Range wildcard. If used as files prefix (`[a..z]*`), non-matching entries are not included.
[$start..$end]
```

## Arrays

Note that arrays in zsh work in slightly different ways, so scripts involving them should be shell-specific.

WATCH OUT!!: Arrays can't be directly exported; there are [workarounds](https://stackoverflow.com/a/21941473/210029)!!

Declaration:

```sh
declare -a $myarray                 # declare an empty array; scope is the innermost (e.g. local if in a function)
local -a $myarray                   # declare a local empty array

[local] myarray=(                   # define a local/global filled array; space/newline is separator
  "entry 1" # comment allowed!
  entry2
)
myarray2=("${myarray[@]}")          # copy an array

# More solid way of creating an array from a string.
# `mapfile` is synonym of `readarray`.
#
# WATCH OUT!! If a substituted process is fallible, use an intermediate file (see [substitutions](#substitutions)).
#
# `-t`: remove trailing delimiter; without, each entry will include it
#
# In order to make the variable local, declare it as local before executing `mapfile`
#
mapfile -td. coordinates < <(echo -n "$1")           # WATCH OUT!! Don't forget `-n` on the echo!!
mapfile -t coordinates < <(printf "a a\nb b")
mapfile -t coordinates < <(perl -pe 'chomp if eof' /path/to/file) # strip (file-)end newline
mapfile -t coordinates <<< "$myvar"                  # WATCH OUT!! Appends newline, so can be used only with newline separator
mapfile -td, coordinates < /path/to/file             # Redirection don't append newlines, so it can be safely used
mapfile -d ''                                        # Use null byte separator; WATCH OUT!! There MUST be a space between the empty string and `-d`!!

IFS=, read -ra coordinates <<< $1    # create via `read`; note how `echo -n` is not required

files=(scripts/*.cnf)                # create from list of files; **must use nullglob shopt!**
```

Operations; indexes are 0-based:

```sh
copy=("${source[@]}")                # copy an array
coordinates+=("abc" "cde")           # append entries/concatenate arrays
coordinates+=("${coord2[@]}")        # concatenate array variables
echo ${coordinates[0]}               # access an array; last entry: -1
unset 'coordinates[1]'               # delete an entry
coordinates[1]=b                     # set an indexed value
echo ${#coordinates[@]}              # size (length)

# Array slicing (see https://stackoverflow.com/a/1336245):
#
# - indexes are zero-based
# - only the first can be negative (WATCH OUT! PREFIX WITH SPACE, due to disambiguation with `${var:-val}`)
# - the first can be blank (defaults to 0)
# - spaces are accepted, and interpolation, too
#
slice=("${coordinates[@]:2}")                           # Ruby Interval [2..]
slice=("${coordinates[@]: -2}")                         # Ruby Interval [-2..]
slice=("${coordinates[@]:2:3}")                         # Ruby Interval [2, 3]
slice=("${coordinates[@]: -2:3}")                       # Ruby Interval [-2, 3]
slice=("${coordinates[@]:2:${#coordinates[@]}-2+1 }")   # Ruby Interval [2, len - start + 1] = [2..-1]
echo "${coordinates[@]:2}"                              # When printing a slice, don't use the round brackets

# WATCH OUT! $@ is insane!! (same applies to $*)
#
echo "${@[1]}"        # NOT SUPPORTED!
echo "${@:1:1}"       # Use this to access a single entry
echo "${@: -1}"       # Use this to access the last entry
echo "${@}"           # DAFUQ!!! Doesn't include $0
echo "${@:0}"         # Includes $0 (Ruby [0..]); WATCH OUT! $0 is the program name
echo "${@::3}"        # WATCH OUT! Includes $0 (Ruby [0, 3])
echo "${@:1}"         # Ruby [1..]
echo "${@:1:${#@}-2}" # Ruby [1..-2] DAFUQ!!! $#@ doesn't count $0, so must account when accessing by a length expression
echo "${*:${#@}}"     # Last element (as string); use `(${@...)` for array

# Simple inclusion tests (there is no direct way)
#
echo "${array[*]}" | grep -qP "\b$value\b"        # easy, if the values don't contain spaces or metachars
printf '%s\0' "${array[@]}" | grep -qz "^$value$" # more rigorous; ensure $value doesn't include metachars.

# Print arrays.
#
printf '%s\n' "${array[@]}"                      # print the entries (one per line); `echo` prints all in one line; WATCH OUT! doesn't
                                                 # work with other parameters
declare -p array                                 # very convenient, although debug-style format

# Joining arrays; slicing is supported.
#
# The pure multi-char bash version is ugly (see https://stackoverflow.com/a/17841619 and https://dev.to/meleu/how-to-join-array-elements-in-a-bash-script-303a).
#
echo $(IFS=,; echo "${arr[*]}")                  # Single-char; !!! don't forget the `;` and the quotes !!!
perl -e 'print join("--", @ARGV)' -- "${arr[@]}" # Multi-char, perl `--` is for safety, if any value starts with `-`
printf %s "${arr[@]/#/->}"                       # Join, but also prepend the separator; works also with `arr[*]` (for interpolation)

# Iteration (regular vars/$@)
#
for entry in "${coordinates[@]}"; do echo $entry; done
for (( i = 0; i < "${#coordinates[@]}"; i++ ))         # arithmetic iteration
for param in "$@"; do echo $param; done
```

### Snippets

Sort an array (/keys of an associative array):

```sh
# For entries with newlinews, `sort -z` and mapfile `-d ''` should be used.
#
mapfile -t sorted_keys < <(printf '%s\n' "${!my_arr[@]}" | sort)

# Sorting more complex entries; relies on entries not including newlines.
#
my_arr=("entry1 key2" "entry2 key1")
mapfile -td$'\n' sorted_arr < <(printf $'%s\n' "${my_arr[@]}" | sort -k 2)
```

Iterate a multiple lines output, assigning it to an array:

```sh
binlogs=$(mysql -u"$my_user" -p"$my_pwd" -h"$my_host" -BNe 'SHOW BINARY LOGS')

# `-r` prevents interpreting the escape character (`\`)
# `-a` splits the tokens (by space) into an array; not supported by Zsh.
#
# WATCH OUT!
#
# - Line surrounding whitespace is trimmed (if `IFS= ` is used, each whole line will be stored
#   in binlog[0])
# - If the last line doesn't have a newline, it's not yielded (!); if it's an empty line, an empty
#   value is yielded. Therefore, consider both the input and the effect of `<<<`; in order not
#   to append a newline, use `< <(echo -n ...)`.
# - If the output program reads from stdin (e.g. ffmpeg), append '< /dev/null' (or use specific
#   program options); see https://www.shellcheck.net/wiki/SC2095
#
while read -r -a binlog; do
  echo Filename: "${binlog[0]}", Position: "${binlog[1]}"
done <<< "$binlogs"

# Version with process substitution (and without array splitting)
# `IFS=` prevents surrounding whitespace trimming
#
while IFS= read -r myvar; do myprocess "$myvar"; done < <(mycommand)
```

## Associative arrays

See array notes in the [related section](#arrays).

See https://www.artificialworlds.net/blog/2012/10/17/bash-associative-array-examples.

```sh
# Quotes inside the brackets are optional

declare -A MYHASH                    # explicit declaration; scope is the innermost (e.g. local if in a function)
declare -A MYHASH=([foo]=1 [bar]=2)  # explicit definition with multiple values; DON'T FORGET `declare -A`!!
[local] MYHASH[foo bar]=baz          # implicit local/global definition (single value only)
unset MYHASH                         # destruction
MYHASH=()                            # empty the hash

# explicit creation from a string; escaping is not required, but shellcheck complains otherwise
content="[foo]=1"
eval declare -A MYHASH=\("$content"\)

echo ${MYHASH[foo]}                  # access an entry
echo ${MYHASH[foo]:-bar}             # empty var substitution works!
unset MYHASH[foo]                    # remove an entry
[[ -v MYHASH[foo] ]]                 # test if a key exists (is defined)

echo ${MYHASH[@]}                    # values (!!!)
echo ${!MYHASH[@]}                   # keys
echo ${#MYHASH[@]}                   # size
declare -p MYHASH                    # print the AA (in declaration form)

# Iterate; in order to safely iterate the sorted keys, use `mapfile` as in the Arrays section.
#
for key in "${!MYHASH[@]}"; do
  val=${MYHASH[$key]}
done

# Unsafe sorted keys iteration.
#
for key in $(printf "%s\n" "${!MYHASH[@]}" | sort); do :; done

# Test if a variable is an associate array (!!)
# `declare -p` prints the declaration; writes to stderr if the variable doesn't exist
# watch out! if a variable was defined via `declare -p`, it will be printed.
#
[[ "$(declare -p MYHASH 2> /dev/null)" == "declare -A"* ]] && echo "is associative array"
```

In order to pass an AA to a function:

```sh
declare -A MYHASH=([foo]=bar)

function myfx {
  local -n hash_ref=$1        # here: copy it from the position param
  hash_ref[baz]=qux
}

myfx MYHASH
echo ${#MYHASH[@]} # 2
```

## Shellcheck

Use `shellcheck disable=all` to disable all checks.

In order to generically solve the `SC1091` issue:

```
source "$(dirname "$0")/included_file.sh"
       ^-- SC1091: Not following: ./included_file.sh: openBinaryFile: does not exist (No such file or directory)
```

run shellcheck with the options:

- `-x`           : allow `source` outside of files
- `-P SCRIPTDIR` : use to script path to look for sourced files

## Script operational concepts/Useful scripts

### Split a string (single/multi-char delimiter) into an array

Reference: https://stackoverflow.com/a/45201229.

Cheap; can use only for unspaced stuff (single-char, or multi without space):

```sh
coordinates=(${1//$delimiter/ }) # split by `.`
```

For a more solid solution, use `mapfile` (see [Arrays section](#arrays)).

Via IFS (single-char):

```sh
# Use IFS to tokenize a string with alternative characters (they're considered a set of **single** chars); **must** unset after:

IFS=",;"
for token in $'a,b c;d\ne'; do echo "<$token>"; done
# <a>
# <b c>
# <d
# e>

unset IFS

# The `while/IFS/read` pattern has different semantics:

while IFS=",;" read -r t0 t1 t2; do echo "<$t0><$t1><$t2>"; done <<< $'a,b c;d\ne'
# <a><b c><d>
# <e><><>
```

Via while + Bash string replace (multi-char):

```sh
# Clever; consumes the input string:

result=()
while [[ -n $input ]]; do
  result+=("${input%%"$delimiter"*}")
  input=${input#*"$delimiter"}
done
```

Split via null char (multi-char) ():

```sh
mapfile -td '' a < <(echo "$string, " | awk '{ gsub(/, /,"\0"); print; }'); unset 'a[-1]'
```

### Ask for input (keypress)

Input one char; if `variable_name` is specified, the input is stored in the variable:

```sh
read -rsn1 [<variable_name>]
```

### Error handling

#### Simulating error bubbling (unexpected `set -o errexit` behavior)

WATCH OUT! This doesn't work as expected with `set -o errexit` (for functions as well):

```sh
{
  echo foo
  false
  echo bar
} || log_error "Error!"
```

it will execute `echo bar` (see https://stackoverflow.com/q/50785352/210029) !!!

Do like this:

```sh
{
  echo foo &&
  false    &&
  echo bar &&
} || log_error "Error!"
```

WATCH OUT!! There may be bug in Bash; the following error exits the function, but not the script!:

```sh
function foo { echo "${@:0:-1}"; echo skipped_here; }

set -o errexit
foo "$@"
echo reaches_here
```

#### Trapping errors (hooks)

Execute a (explicit) command on exit.

`EXIT` will trap everything (which can be fine), so don't use it with `ERR`/`INT`, otherwise the handler will be invoked once for each signal.

WATCH OUT!

- Trapping `EXIT` and `ERR`/`INT` will cause multiple traps
- Can't register multiple handlers for the same signal; the last will override the preceding
  - However, each subshell (script) has their own handlers.
- If both `ERR` and `INT` are trapped, in some cases, both will trigger!
  - If want to trap exits with error, use `trap '[[ $? -ne 0 ]] && _error_exit_hook' EXIT`

```sh
LOCKFILE=/var/lock/makewhatis.lock

# Explicit command form. WATCH OUT! Use single outer quotes, otherwise the content is interpolated!
# The context is still valid, so variables in the string will be referenced.

trap '{ rm -f $LOCKFILE; }' EXIT

# Function form:
#
function remove_lockfile {
  rm -f "$LOCKFILE"
}

trap remove_lockfile EXIT

# It's possible to nest functions (!); WATCH OUT!! The hooked function can't access local variables when it's invoked!
#
function register_exit_hook {
  # don't make it local, otherwise it won't be available to _exit_hook()!!!
  LOCKFILE="/tmp/mylockfile"
  function _exit_hook { rm -f "$LOCKFILE"; }
  trap _exit_hook EXIT
}
```

### Log a script output/Enable debugging (log)

```sh
# Send stdout/stderr also to a logfile.
# `-i`: don't terminate immediately tee in case of interrupt signal (tee will regardless terminate when
# the script exits).
# WATCH OUT! Without `-a` (append), in at least one case, a part (the top) of the log was missing.
#
rm -f "$logfile"
exec > >(tee -ai "$logfile") 2>&1

# Debugging log; includes stdout/stderr content (see above).
#
function setup_logging {
  exec 5> "$logfile"
  BASH_XTRACEFD=5
  rm -f "$logfile"
  exec > >(tee -ai "$logfile") 2>&1
  set -x
}

# Make debugging log more informative.
export PS4='+ ${BASH_SOURCE:-}:${FUNCNAME[0]:-}:L${LINENO:-}:   '
```

The `:` command can be used to add pseudo-comments that are displayed in the debug log:

```sh
: this comment appears in the log
my_operation
```

See [here](https://johannes.truschnigg.info/writing/2021-12_colodebug) for a more sophisticated version.

### Print to stdout while setting a variable

```sh
var=$(cmd | tee /dev/tty)
```

### Check if there's data in stdin

Check if there is data in stdin (can't be done in Bash natively; see https://unix.stackexchange.com/q/33049):

```sh
first_byte=$(dd bs=1 count=1 2>/dev/null | od -t o1 -A n | tr -dc 0-9)

if [[ -z $first_byte ]]; then
  # stuff to do if the input is empty
else
  {
    printf "\\$first_byte"
    cat
  } | {
    # stuff to do if the input is not empty
  }
fi
```

### Check if a script is `source`d

```sh
# true if script is sourced (BASH_SOURCE is the file including the statement)
[[ ${BASH_SOURCE[0]} != "$0" ]]
```

### Sudo-related tasks

Workaround sudo expiry during a long-running script ([source](https://serverfault.com/a/702019)):

```sh
sudo -v

while true; do
  sleep 60
  # `kill -0`: doesn't send any actual signal - used (here) to check if the process exists
  # `$$`: parent script (script running this snippet)

  kill -0 "$$" || exit

  # `-n`: non-interactive
  sudo -nv
done 2>/dev/null &
```

Check if a script is run as sudo:

```sh
if [[ $EUID -eq 0 ]]; then
  >&2 echo "Don't run this script as root!"; exit 1
fi
```

Switch to root user inside a script; this is not possible, but there is a workaround. This is convenient especially when the file is in the sudoers:

```sh
function ensure_sudo_invocation {
  if [[ $(id -u) -ne 0 ]]; then
    # Avoids having to call `sudo`.
    #
    # WATCH OUT! Doesn't export the environment. Use `-E` if needed (and supported).
    # `sudo env K=V...` can't be used, unless /usr/bin/env is in the sudoers
    # Another workaround is to dump the variables into a temp file and source it.
    #
    sudo "$0" "$@"
    exit $?
  else
    # If other scripts in the invoking user are required, must manually add the $PATH.
    export PATH=$c_scripts_path:$PATH
  fi
}

ensure_sudo_invocation "$@"
```

Graphical sudo:

```sh
pkexec env DISPLAY="$DISPLAY" XAUTHORITY="$XAUTHORITY" "$0" "$@"
```

### Time-related functionalities (incl. benchmarking)

```sh
# Check time passed, using for loop!
# $SECONDS can be used in general, and it will start ticking immediately after set.
#
for ((SECONDS=0; SECONDS < 10; )); do sleep 1; done

# Wait until next beginning of second
sleep 0.$(printf '%04d' $((10000 - 10#$(date +%4N))))

# Countdown timer
for i in $(seq $((3600 * 4)) -1 1); do echo -ne "\r$i "; sleep 1; done
```

When using `time`, must be careful to what is used (built-in vs. `/usr/bin/time`). The builtin has milliseconds accuracy, but setting it is d*cking confusing.  
Technically, `time` is a Bash keyword (!). If a variable is set on the same command, instead of the keyword, `/usr/bin/time` is used:

```sh
# ok
TIMEFORMAT='Result: %3R'
time mycommand

# NO!!!!
TIMEFORMAT='Result: %3R' time mycommand
```

### Parse commandline options (`getopt`)

See http://www.bahmanm.com/blogs/command-line-options-how-to-parse-in-bash-using-getopt for further help (eg. optional args).

Note that when using this getopt (GNU) inside a function, all the processing should be performed inside the function - `$@/$#` are not as expected after returning.

```sh
c_help="Usage: $(basename "$0") [-h|--help] ..."

# Options (case sensitive): [-h|--help], [-s|--shared-folders], [-c|--cd1-image MANDATORY_ARGUMENT]
# `--name` is the name of the program printed when an error is reported.
#
# If not in a function, the simplest approach is:
#
#     if params=$(getopt ...); then exit 1; fi
#     eval ...
#     unset params
#
local params
params=$(getopt --options hsc: --long help,shared-folders,cd1-image: --name "$(basename "$0")" -- "$@")

eval set -- "$params"

# DON'T FORGET THE `shift` commands and the `--` case.
# Rigorously, one should add the '*' case (internal error), but it's not required.
#
while true; do
  case $1 in
    -h|--help)
      echo "$c_help"
      exit 0 ;;
    -s|--shared-folders)
      USE_SHARED_FOLDERS=1
      shift ;;
    -c|--cd1-image)
      CD1_IMAGE=$2
      shift 2 ;;
    --)
      shift
      break ;;
  esac
done

if [[ $# -ne 0 ]]; then
  echo "$c_help"
  exit 1
fi
```

Non-option params are available, after the `while` block, from `$1`.

### Poor man's configfile parser

Source (modified): https://stackoverflow.com/a/20815951.

Supports empty lines and single-line comments; spaces around the equal sign are trimmed.

```sh
while IFS='= ' read -r key value || [[ -n $key ]]; do   # `-n`: include the last line, if it hasn't a newline
  if [[ -n $key && ! $key =~ ^\ *# ]]; then
    declare -g "$key"="$value"                          # `-g`: declare as global (otherwise, it's local); add `-x` to also export them
  fi
done < "$configuration_file"
```

Convenient extra functionalities:

```sh
    # Interpret values starting with `$` as references to other variables
    if [[ $value == \$* ]]; then
      eval declare -g $key=$value
    # Interpret values starting with `(` as arrays
    elif [[ $value == '('* ]]; then
      eval declare -ga "$key"="$value"
    else
      declare -g $key="$value"
    fi
```

### `bash` command options

```sh
# Pass arguments to bash command, when the script is passed from stdin.
#
echo 'echo $1' | bash -s 1
```

### Variables printing function

Invoke as `print_variables varname1 varname2 ...`.

```sh
function print_variables {
  for variable_name in "$@"; do
    declare -n variable_reference="$variable_name"

    echo -n "$variable_name:"

    case $(declare -p "$variable_name") in
    "declare -a"* )
      for entry in "${variable_reference[@]}"; do
        echo -n " \"$entry\""
      done
      ;;
    "declare -A"* )
      for key in "${!variable_reference[@]}"; do
        echo -n " $key=\"${variable_reference[$key]}\""
      done
      ;;
    * )
      echo -n " $variable_reference"
      ;;
    esac

    echo
  done

  echo
}
```

### Current shell/OS

Simplest, from terminal: `echo $0`. Don't use `$SHELL`, as it can refer to the parent.

Os: `$OSTYPE` -> `linux-gnu`, `darwin21.0`.

### Convert associative array to JSON

From [here](https://stackoverflow.com/q/44792241):

```sh
for key in "${!hash[@]}"; do
    printf '%s\0%s\0' "$key" "${hash[$key]}"
done |
  jq -Rs '
    split("\u0000")
    | . as $a
    | reduce range(0; length/2) as $i
        ({}; . + {($a[2*$i]): ($a[2*$i + 1]|fromjson? // .)})'
```

WATCH OUT!!: There is no way to directly support null.

## Pitfalls

- `git log --oneline | head -n 1`: will fail due to `git` raising an error when the pipe is closed
  - can use `awk 'NR <= N { print }'` or `perl -ne 'print if $. <= N'`

## Shell colors

See https://www.lihaoyi.com/post/BuildyourownCommandLinewithANSIescapecodes.html.

Ruby convenience script:

```ruby
STANDARD_FOREGROUND_COLORS = [
  BLACK_FG = "\u001b[30m",
  RED_FG = "\u001b[31m",
  GREEN_FG = "\u001b[32m",
  YELLOW_FG = "\u001b[33m",
  BLUE_FG = "\u001b[34m",
  MAGENTA_FG = "\u001b[35m",
  CYAN_FG = "\u001b[36m",
  WHITE_FG = "\u001b[37m",
]

BRIGHT_FOREGROUND_COLORS = [
  BRIGHT_BLACK_FG = "\u001b[30;1m",
  BRIGHT_RED_FG = "\u001b[31;1m",
  BRIGHT_GREEN_FG = "\u001b[32;1m",
  BRIGHT_YELLOW_FG = "\u001b[33;1m",
  BRIGHT_BLUE_FG = "\u001b[34;1m",
  BRIGHT_MAGENTA_FG = "\u001b[35;1m",
  BRIGHT_CYAN_FG = "\u001b[36;1m",
  BRIGHT_WHITE_FG = "\u001b[37;1m",
]

# The bright background versions make the foreground color bright (!).
#
BACKGROUND_COLORS = [
  BLACK_BG = "\u001b[40m", # good with BRIGHT_WHITE_FG
  RED_BG = "\u001b[41m",
  GREEN_BG = "\u001b[42m",
  YELLOW_BG = "\u001b[43m",  # good with [BRIGHT_]BLACK_FG
  BLUE_BG = "\u001b[44m",
  MAGENTA_BG = "\u001b[45m",
  CYAN_BG = "\u001b[46m",
  WHITE_BG = "\u001b[47m",
]

RESET_COLOR = "\e[0m"

def colorize_warning_thresholds(value, red_threshold, yellow_threshold)
  if value < red_threshold
    prefix, suffix = "#{BRIGHT_WHITE_FG}#{RED_BG}", RESET_COLOR
  elsif value < yellow_threshold
    prefix, suffix = "#{BRIGHT_BLACK_FG}#{YELLOW_BG}", RESET_COLOR
  end

  "#{prefix}#{value}#{suffix}"
end

def print_color_combinations
  BACKGROUND_COLORS.each do |bg_color|
    STANDARD_FOREGROUND_COLORS.each do |fg_color|
      print "#{bg_color}#{fg_color} 90 #{RESET_COLOR}"
    end

    print "   "

    BRIGHT_FOREGROUND_COLORS.each do |fg_color|
      print "#{bg_color}#{fg_color} 90 #{RESET_COLOR}"
    end

    puts "", ""
  end

  nil
end
```
