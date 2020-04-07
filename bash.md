# Bash

- [Bash](#bash)
  - [Shellopts](#shellopts)
  - [Test conditions](#test-conditions)
    - [Regular expressions](#regular-expressions)
    - [Applications](#applications)
  - [String operations](#string-operations)
    - [Examples](#examples)
  - [Arithmetic operations](#arithmetic-operations)
  - [Date operations](#date-operations)
  - [Redirections](#redirections)
  - [Proces substitution](#proces-substitution)
  - [Ask for input (keypress)](#ask-for-input-keypress)
  - [Script operational concepts](#script-operational-concepts)
    - [Trapping errors](#trapping-errors)
    - [Log a script output/Enable debugging [log]](#log-a-script-outputenable-debugging-log)
    - [Switching to root user inside a script](#switching-to-root-user-inside-a-script)
    - [Check if there's data in stdin](#check-if-theres-data-in-stdin)
    - [Check if a script is `source`d](#check-if-a-script-is-sourced)
    - [Workaround sudo expiry during a long-running script](#workaround-sudo-expiry-during-a-long-running-script)
    - [Interesting/useful time-related functionalities](#interestinguseful-time-related-functionalities)

## Shellopts

```sh
set -o errtrace           # `-E`: trap errors also inside functions
set -o xtrace             # `-x`: debugging mode; prints all the statements
shopt -s nocasematch      # case insensitive matches
```

**important!** don't forget to set `errtrace` when trapping errors!

## Test conditions

```
-z string                 string is empty
-n string                 string is not empty

-e <filename>             file/directory/symlink exists; symlink is followed before checking!
-f <filename>             file exists
-x filename               file exists and it's executable
-d <dirname>              directory exists
-L <filename>             file/directory is a symlink
-b <filename>             file is a block device

! -s <filename>           file is empty

-t 0                      input is terminal (false if stdin)
```

### Regular expressions

!! WATCH OUT !!

1. They're not entirely perl-compatible (see http://tldp.org/LDP/abs/html/x17129.html)
2. The expressions are not quoted
3. Use a temporary variable (or cmd subsitution) when using slashed metacharacters (see https://stackoverflow.com/a/12696899)
4. `^` and `$` refer to the *whole string*, not the line
5. DON'T WRAP with ruby regex slashes!!!

Metacharacters:

- `\w`  -> `[[:alnum:]]`
- `\b`  -> `\b`

Examples:

```sh
[[ 192.168.1.2 =~ ^[0-9]{1,3}(\.[0-9]{1,3}){3}$ ]] && echo MATCHES
re='\bword\b'; [[ ' word ' =~ $re ]] && echo MATCHES
[[ ' word ' =~ $(echo '\bword\b') ]] && echo MATCHES
```

### Applications

Check if a binary/executable is in PATH/exists (**don't** use which):

```sh
[[ -x "$(command -v <command>)" ]]
```

Execute a command if a process is not running:

```sh
! pgrep -fa mysqld && (mysqld &)                       # Zsh doesn't need brackets for this semantics
if [[ -z "$(pgrep -fa mysqld)" ]]; then mysqld & fi
```

## String operations

Substitutions generally have the single (single operator) or global (double operator) version; where this is the case, the global version is presented.

Parameter expansion supports internal expansion, and escape chars: `line=${line//$'\t'/ }`

```sh
${str:-<expr>}                    # default a parameter: set if undefined or blank
${str:+<expr>}                    # if the var is set, replace with <expr> (which can include the $param itself!)

${str//search/replace}            # substitution (!! NOT REGEX !!) ->
${str//[a-e]/ }                   # -> but supports at least character classes!

${str:[<start>][:<end_expr>]}     # substring (0-based)
                                  # `start`: blank: first char
                                  # `end_expr`: positive: length; negative: position referring to last (!! -1 = BEFORELAST !!); blank: until end

${str^^}                          # upper case (single `^` applies only once)
${str,,}                          # down case (single `^` applies only once)
```

(g)sub (applied to filenames):

```sh
${filename%<.ext>}                # strip specific <ext>ension; `*` can be used
${filename%%.*}                   # strip any extension (anything after the first dot)
${filename##*.}                   # extract (last) extension
${filename##*/}                   # extract basename, eg. `"${PWD##*/}"`
```

### Examples

```sh
${str:1}                          # substring: remove the first charater
${str::-1}                        # substring: remove the last character
```

## Arithmetic operations

```sh
(( <expr> ))                      # if <expr> == 0, evaluates to false!
let param=<expr>                  # if <expr> == 0, evaluates to false!
```

**Important**: arithmetic operations will cause exit if they evaluate to 0 and the `errexit` shellopt is set:

```sh
(( val ))                         # exits if `val` is 0
(( val++ ))                       # postfix exits if the old `val` value is 0
param=$(( val++ ))                # this doesn't exit
```

## Date operations

Add/subtract dates: convert to seconds, then perform an operation via `date --date`, then convert back:

```sh
timestamp_in_secs=`date -d "2015-12-12 13:13:28" +"%s"`
seconds_to_subtract=90
formatted_result=`date '+%Y-%m-%d %H:%M:%S' --date="@$((timestamp_in_secs – seconds_to_subtract))"`
```

## Redirections

```sh
&> "$filename"                                          # redirect both stdout and stderr to <filename>
>&2 echo "error"                                        # write to stderr

{ mycommand 3>&2 2>&1 1>&3 | grep -v "skipme" >&3; } 3>&2 2>&1  # filter out stderr message

exec 200> "$filename"                                   # associate a file to a file descriptor (create if not existing)
```

## Proces substitution

Creates a file (descriptor) from the command.

For input file; typically used to feed output of multiple commands to another command:

```sh
meld <(command_list_1) <(command_list_2)>
```

For output file; typically used with tee, to both print to stdout and redirect to a pipeline:

```sh
# If we do `| tee | xsel -ib`, the output is not printed!
#
echo "abc" >(xsel -ib)
```

## Ask for input (keypress)

Input one char; if `variable_name` is specified, the input is stored in the variable:

```sh
read -rsn1 [<variable_name>]
```

## Script operational concepts

### Trapping errors

Execute a (explicit) command on exit; generally convenient to trap ERR and INT:

```sh
LOCKFILE=/var/lock/makewhatis.lock

# Explicit command form:
#
trap "{ rm -f $LOCKFILE; }" EXIT

# Function form:
#
function remove_lockfile {
  rm -f "$LOCKFILE"
}

trap remove_lockfile EXIT
```

### Log a script output/Enable debugging [log]

```sh
# General log
#
exec > >(tee -i "$logfile")
exec 2>&1

# Debugging log
#
exec 5> "$logfile"
BASH_XTRACEFD="5"
set -x
```

### Switching to root user inside a script

This is not possible, but there is a workaround:

```sh
if [[ $(id -u) -ne 0 ]]; then
  sudo "$0" "$@"
  exit $?
fi
```

### Check if there's data in stdin

Check if there is data in stdin (can't be done in Bash natively; see https://unix.stackexchange.com/q/33049):

```sh
first_byte=$(dd bs=1 count=1 2>/dev/null | od -t o1 -A n | tr -dc 0-9)

if [ -z "$first_byte" ]; then
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
[[ "${BASH_SOURCE[0]}" != "$0" ]] # true if script is sourced
```

### Workaround sudo expiry during a long-running script

`$$` is the parent script, which is the script running this snippet.

```sh
sudo -v

while true; do
    sudo -n true
    sleep 60
    kill -0 "$$" || exit
done 2>/dev/null &
```

### Interesting/useful time-related functionalities

```sh
# Check time passed.
SECONDS=0; while [[ $SECONDS -lt 10 ]]; do sleep 1; done

# Wait until next beginning of second
sleep 0.$(printf '%04d' $((10000 - 10#$(date +%4N))))
```
