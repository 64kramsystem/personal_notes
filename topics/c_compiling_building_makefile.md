# Makefile

- [Makefile](#makefile)
  - [Make](#make)
    - [Useful stuff](#useful-stuff)
    - [Sample breakdown](#sample-breakdown)
      - [Variables](#variables)
      - [Targets/Symbols](#targetssymbols)
      - [Shell commands execution](#shell-commands-execution)
      - [If/then/else](#ifthenelse)
  - [CMake](#cmake)
  - [General build concepts](#general-build-concepts)
    - [Find compilation commands](#find-compilation-commands)
    - [Build errors](#build-errors)
    - [`glibconfig.h: No such file or directory`](#glibconfigh-no-such-file-or-directory)
      - [`cannot find install-sh, install.sh, or shtool in ...`](#cannot-find-install-sh-installsh-or-shtool-in-)

## Make

### Useful stuff

Print variables during run (put this in a target):

```makefile
echo [CC: ${CC}] [LIBS: ${LIBS}] # etc.
```

```sh
# General tasks:
#
# Note that 'make' depends on the product of './configure'.
#
make clean           # Remove `make` output
make distclean       # Remove `configure`+`make` output

# Print make commands (dry run)
#
make -n
```

### Sample breakdown

The makefile name can be `makefile` or `Makefile`

This makefile assumes the file `main.c` and the dependencies `module.c`/`module.h`.

#### Variables

Variables are conventionally upper case, and can be referenced with curly braces; they are referenced with `${VAR_NAME}`; missing vars produce an empty string.

- `:=` : "simple assignment"; it's evaluated only once;
- `=`  : "recursive assigment"; it's re-evaluated every time (so changing the value is recognized);
- `?=` : assign if not already done;
- `+=` : append;

Variables can also be used in the dependencies part.

```makefile
CC := gcc
OPTIONS := -O2 -g -Wall
INCLUDES := -I .
LIST_FILES := ls -l
```

Set a variable if blank:

```sh
HOSTCC := $(if $(HOSTCC),$(HOSTCC),$(CC))
```

Variables can be overriden from the commandline by passing them as `make` params:

```sh
make "LDFLAGS=-static" CC="riscv64-unknown-linux-gnu-gcc -I/path/to/riscv-gnu-toolchain/riscv-gcc/zlib -L/path/to/riscv-gnu-toolchain/riscv-gcc/zlib"
```

#### Targets/Symbols

Targets which are filenames, are compiled only if the file changed; non-filenames are always compiled.
A target is a filename (or an arbitrary name), followed by the dependencies; each target is executed in a separate subshell.

An action can be prefixed with `-` to ignore errors (see clean).

The prefix `@` doesn't echo the action.

`%` : wildcard.

Variables:

- `$@` : (full target name of the current target)
- `$?` : (returns the dependencies that are newer than the current target)
- `$*` : (returns the text that corresponds to % in the target)
- `$<` : (name of the first dependency)
- `$^` : (name of all the dependencies with space as the delimiter)

```makefile
# Specify that those targets are not filenames - in this case, this is optional.
# .PHONY can accept multiple values.
#
.PHONY all

# `all` is the default target
#
all: main.o module.o
	${CC} ${OPTIONS} main.o module.o -o target_bin

# The following are generalized in the target with wildcard.
#
# main.o: main.c module.h
# 	${CC} ${OPTIONS} ${INCLUDES} -c main.c
# module.o: module.c module.h
# 	${CC} ${OPTIONS} ${INCLUDES} -c module.c

%.o: %.c %.h
	${CC} ${OPTIONS} ${INCLUDES} -c $*.c
```

#### Shell commands execution

`$(shell <command>)`: Executes a shell command (eg. `$(shell ls)`); it can also be assigned to a variable (eg. `VAR := $(...)`).

A variable can be used a command (it seems that it must be the only content of the action) - see this example.

```makefile
.PHONY: list
list:
	@$(LIST_FILES)

.PHONY: clean
clean:
	rm -f *.o
	-rm target_bin
```

#### If/then/else

- `if(condition, true_branch, false_branch)`
- `ifeq($(VARIABLE),value)`
- `ifneq`

```makefile
# Also shows recursive rules invocation (`$(MAKE)`)
#
# Also shows checking if a variable is defined, and raising errors.
#
.PHONY: install
install:
ifeq($(GOBY_BINPATH),)
	go install $(RELEASE_OPTIONS) .
else
ifndef GOBY_LIBPATH
	$(error GOBY_BINPATH requires GOBY_LIBPATH to be set)
endif
	$(MAKE) build
	mkdir -p ${DESTDIR}${GOBY_BINPATH}
	mv -v goby ${DESTDIR}${GOBY_BINPATH}
	mkdir -p ${GOBY_LIBPATH}
	cp -R lib ${GOBY_LIBPATH}
endif
```

## CMake

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

## General build concepts

Configure compilation:

- `-static`         : compile statically
- `-Dmacro[=value]` : define a macro (e.g. for `ifdef`); the option is cumulative

### Find compilation commands

There are a few options:

- CMake can generate the compile commands: `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1`, which outputs to `compile_commands.json`
- [intercept-build](https://github.com/immunant/c2rust#-with-intercept-build)
- [build-logger](https://github.com/Ericsson/codechecker/tree/master/analyzer/tools/build-logger)
  - this also includes linking commands
- `bear` tool, which generates the compile options for each file (`bear make`)

Alternative: wrap the compiler and linker in proxy scripts that log the commands 🤓.

Background [here](https://immunant.com/blog/2020/01/quake3).

### Build errors

### `glibconfig.h: No such file or directory`

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

#### `cannot find install-sh, install.sh, or shtool in ...`

Execute:

```sh
autoreconf -vif
# follow up with ./configure etc.
```
