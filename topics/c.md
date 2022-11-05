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
  - [Common issues](#common-issues)
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

- `char`            : i8
- `[unsigned] short`: 2 bytes
- `[unsigned] int`  : 2/4 bytes (typically 4)
- `[s]size_t`       : alias; approximately Rust usize/isize (`s`igned)

### Base functions

#### printf

| types        | unsigned | signed  | note                  |
| ------------ | :------: | :-----: | --------------------- |
| int, boolean |   `u`    | `d`/`i` |                       |
| long         |   `lu`   |  `ld`   |                       |
| pointer      |   `p`    |    -    | must cast to `void*`! |
| size_t       |   `zu`   |  `zd`   |                       |
| string       |   `s`    |    -    |                       |

For kernel-related types, see the [kernel programming notes](../books_courses/linux_kernel_programming.md#logging).

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

## Common issues

See also [build errors](c_compiling_building_makefile.md#build-errors).

### Error `multiple definition of '<variable>'` (linker)

Make the variable `static`.
