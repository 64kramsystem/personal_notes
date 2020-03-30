## Table of contents

- [Table of contents](#table-of-contents)
- [Shellopts](#shellopts)
- [Test conditions](#test-conditions)
  - [Regular expressions](#regular-expressions)
  - [Applications](#applications)
- [String operations](#string-operations)
  - [Examples](#examples)
- [Arithmetic operations](#arithmetic-operations)
- [Redirections](#redirections)
- [Trapping errors](#trapping-errors)

## Shellopts

```sh
set -o errtrace           # `-E`: trap errors also inside functions
set -o xtrace             # `-x`: debugging mode; prints all the statements
shopt -s nocasematch      # case insensitive matches
```

**important!** don't forget to set `errtrace` when trapping errors!

## Test conditions

```
-z string 								string is empty
-n string								  string is not empty
-e <filename>							file/directory/symlink exists; symlink is followed before checking!
-f <filename>							file exists
-x filename								file exists and it's executable
-d <dirname>							directory exists
-L <filename>							file/directory is a symlink
-b <filename>             file is a block device
! -s <filename>						file is empty
```

### Regular expressions

**Watch out!!**

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

```sh
${str:-<expr>}                    # default a parameter: set if undefined or blank
${str:+<expr>}                    # if the var is set, replace with <expr> (which can include the $param itself!)

${str:[<start>][:<end_expr>]}     # substring (0-based)
                                  # `start`: blank: first char
                                  # `end_expr`: positive: length; negative: position referring to last (!! -1 = beforelast !!); blank: until end
```

(g)sub (applied to filenames):

```sh
${filename%<.ext>}                # strip specific <ext>ension; `*` can be used
${filename%%.*}                   # strip any extension (anything after the first dot)
${filename##*.}                   # extract extension
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

## Redirections

```sh
&> <filename>                                          # redirect both stdout and stderr to <filename>
>&2 echo "error"                                       # write to stderr
{ mycommand 3>&2 2>&1 1>&3 | grep -v "^Message to skip" >&3; } 3>&2 2>&1  # filter out stderr message

exec 200> <filename>                                   # associate a file to a file descriptor (create if not existing)
```

## Trapping errors

Execute a (explicit) command on exit; generally convenient to trap ERR and INT:

```sh
LOCKFILE=/var/lock/makewhatis.lock

# Explicit command form:
#
trap "{ rm -f $LOCKFILE; }" EXIT

# Function form:
function remove_lockfile {
  rm -f "$LOCKFILE"
}

trap remove_lockfile EXIT
```
