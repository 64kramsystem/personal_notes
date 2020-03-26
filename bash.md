## Table of contents

- [Test conditions](#test-conditions)
  - [Applications](#applications)
- [Redirections](#redirections)
- [String operations](#string-operations)

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

## Redirections

```sh
&> <filename>                                          # redirect both stdout and stderr to <filename>
>&2 echo "error"                                       # write to stderr
{ mycommand 3>&2 2>&1 1>&3 | grep -v "^Message to skip" >&3; } 3>&2 2>&1  # filter out stderr message

exec 200> <filename>                                   # associate a file to a file descriptor (create if not existing)
```

## String operations

```sh
${param:-<expr>} 				                               # default a parameter: set if undefined or blank
${param:+<expr>}                                       # if the var is set, replace with <expr> (which can include the $param itself!)

${param:<start>[:<end>]}                               # substring (0-based); end is included
```
