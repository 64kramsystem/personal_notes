# C

- [C](#c)
  - [Requirements](#requirements)
  - [Basics](#basics)
    - [Base program](#base-program)
    - [Data types](#data-types)
    - [Base functions](#base-functions)
      - [printf](#printf)
  - [Preprocessor](#preprocessor)
  - [Stdlib](#stdlib)
    - [Env variables](#env-variables)
  - [Linux](#linux)
    - [Headers](#headers)
    - [User ids](#user-ids)
  - [Conveniences](#conveniences)
    - [Print a struct instances's bytes](#print-a-struct-instancess-bytes)
    - [Print a stack trace on segfault](#print-a-stack-trace-on-segfault)
    - [Find the current executable filename](#find-the-current-executable-filename)
    - [Build a C program's call graph](#build-a-c-programs-call-graph)
  - [Compiler/builder stuff](#compilerbuilder-stuff)
    - [CMake](#cmake)
    - [Find compilation commands](#find-compilation-commands)
  - [Common issues](#common-issues)
    - [Error `glibconfig.h: No such file or directory`](#error-glibconfigh-no-such-file-or-directory)
    - [Error `cannot find install-sh, install.sh, or shtool in ...`](#error-cannot-find-install-sh-installsh-or-shtool-in-)
    - [Error `multiple definition of '<variable>'` (linker)](#error-multiple-definition-of-variable-linker)

## Requirements

In all the examples (stdlib, linux...), the following inclusions are implied:

```c
#include <stdio.h>   // printf(3)
#include <stdlib.h>  // exit(3), EXIT_FAILURE, system(3)
```

The exit with error examples are intentionally simplified (no braces, messages, etc.).

## Basics

### Base program

```c
int main(int argc, char **argv) {
  if (argc != 2) {
    fprintf(stderr, "Invalid number of command line arguments.\n");
    exit(EXIT_FAILURE);
  }

  printf("123");

  system("ls -l /tmp");
}
```

### Data types

- `char`: i8
- `short`: i16

### Base functions

#### printf

| types        | unsigned | signed  | note                  |
| ------------ | :------: | :-----: | --------------------- |
| int, boolean |   `u`    | `d`/`i` |                       |
| long         |   `lu`   |  `ld`   |                       |
| pointer      |   `p`    |    -    | must cast to `void*`! |
| string       |   `s`    |    -    |                       |

## Preprocessor

```c
// Parentheses around the const are valid
#
#ifdef MYCONST
#endif // MYCONST

// Or (`||`) is also supported
//
#if defined MYCONST && defined MYCONST2
#endif
```

## Stdlib

### Env variables

```c
#include <stdlib.h>  // getenv(3)

char *sudo_user = getenv("SUDO_USER");

if (!sudo_user) exit(EXIT_FAILURE);

printf("%s\n", sudo_user);
```

## Linux

### Headers

Located under `/usr/include/linux`.

Other system-dependent includes may be found under `/usr/include/x86_64-linux-gnu/` (e.g. for `#include <sys/types.h>`)

### User ids

```c
#include <pwd.h>        // getpwuid(3)
#include <unistd.h>     // geteuid(2)
#include <sys/types.h>  // uid_t

uid_t euid = geteuid();
struct passwd *passwd = getpwuid(euid);

if (!passwd) exit(EXIT_FAILURE);

printf("%s\n", passwd->pw_name);
```

## Conveniences

### Print a struct instances's bytes

```c
for (int i = 0; i < sizeof(struct fanotify_event_metadata); i++)
  printf("%02x ", ((unsigned char*)metadata)[i]);
printf("\n");
```

### Print a stack trace on segfault

Requires enabling the debug symbols:

```c
#include <execinfo.h>
void sav_handler(int sig);

void sav_handler(int sig) {
  void *array[10];

  // get void*'s for all entries on the stack
  size_t size = backtrace_symbols(array, 10);

  fprintf(stderr, "Error: signal %d:\n", sig);
  backtrace_symbols_fd(array, size, STDERR_FILENO);

  exit(1);
}

int main(int argc, char **argv) {
  signal(SIGSEGV, sav_handler); // install the handler
  raise(SIGSEGV);
}
```

Send the result to addr2line:

```sh
a.out 2>&1 | perl -ne '/\(\+(0x\w+)\)/ && print("$1 ")' | xargs addr2line -e a.out
```

### Find the current executable filename

This is a simple and good enough solution, but for considerations about a very solid solution, see https://stackoverflow.com/a/34271901:

```c
char exe_filename[PATH_MAX];
ssize_t len = readlink("/proc/self/exe", exe_filename, sizeof(exe_filename));
if (len == -1 || len == sizeof(exe_filename)) {
    len = 0;
}
exe_filename[len] = '\0';
```

This is the equivalent of `_pgmptr`/`GetModuleFileName` on Windows.

### Build a C program's call graph

- Download [`egypt`](https://www.gson.org/egypt/egypt.html)
- Add the option `-fdump-rtl-expand` to the CFLAGS
- Compile
- Run `egypt *.expand | tee graph.gv | dot -T pdf -o graph.pdf`

Useful options:

- `egypt --callees fn1,fn2...`: show only the given functions and their calless
- `egypt --callers fn1,fn2...`: show only the given functions and their callers
- `dot -Grotate=90`           : make a L->R graph

VSC has a plugin to display Graphviz files; dotty is terrible; PDF format is good enough.

Other help [here](https://www.gson.org/egypt/egypt.html).

## Compiler/builder stuff

Configure compilation:

- `-static`         : compile statically
- `-Dmacro[=value]` : define a macro (e.g. for `ifdef`); the option is cumulative

### CMake

```sh
# Set variables via `-D`.

# Specify the compiler:
#
cmake -D CMAKE_C_COMPILER=$c_compiler_full_path -D CMAKE_CXX_COMPILER=$cpp_compiler_full_path $project_path

# To add extra CFLAGS:
#
sed -i '/^project/ a set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fdump-rtl-expand")' CMakeLists.txt

# Clean (`cmake --target clean`) doesn't really work, as many commands may be issued. Best thing is
# to clean the repo.
#
rm .gitignore
git clean -fd .
git checkout .gitignore
```

### Find compilation commands

There are a few options:

- CMake can generate the compile commands: `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1`, which outputs to `compile_commands.json`
- [intercept-build](https://github.com/immunant/c2rust#-with-intercept-build)
- [build-logger](https://github.com/Ericsson/codechecker/tree/master/analyzer/tools/build-logger)
  - this also includes linking commands
- `bear` tool, which generates the compile options for each file (`bear make`)

Alternative: wrap the compiler and linker in proxy scripts that log the commands 🤓.

Background [here](https://immunant.com/blog/2020/01/quake3).

## Common issues

### Error `glibconfig.h: No such file or directory`

See https://github.com/dusty-nv/jetson-inference/issues/6:

```sh
sudo ln -s /usr/lib/x86_64-linux-gnu/glib-2.0/include/glibconfig.h /usr/include/glib-2.0/
```

or try this:

```sh
gcc pkg-config --cflags glib-2.0 test.c pkg-config --libs glib-2.0
```

or suggested include path (maybe should be `/usr/lib/x86_64-linux-gnu/glib-2.0/include/glibconfig.h`?):

```
${workspaceFolder}/** /usr/include/glib-2.0/**
```

### Error `cannot find install-sh, install.sh, or shtool in ...`

Execute:

```sh
autoreconf -vif
# follow up with ./configure etc.
```

### Error `multiple definition of '<variable>'` (linker)

Make the variable `static`.
