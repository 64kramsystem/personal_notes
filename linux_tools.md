# Linux tools

- [Linux tools](#linux-tools)
  - [Find](#find)
    - [Examples](#examples)
      - [Search text inside multiple PDFs](#search-text-inside-multiple-pdfs)
  - [Processes](#processes)
    - [Parallel execution](#parallel-execution)
      - [Using GNU Parallle](#using-gnu-parallle)
      - [Using xargs](#using-xargs)
  - [Dates](#dates)
    - [Formatting](#formatting)
    - [Operations](#operations)

## Find

```sh
! (condition)                     # negates a condition
-name xx -not -name yy            # negate search condition
-name "*.c" -o -name "*.h"        # simple OR condition
\( -name xx -or -name yy \)       # OR condition with brackets

-[i]name <pattern>                # use quotes when using wildcards in pattern; case [i]nsensitive
-wholename <value>						    # include the directory name (eg. `.git/config`)
-regextype awk -[i]regex <regex>	# same as name, but with regex; must match the whole name; !! specify the regextype (the default is minimal) !!
						                      # !! uses crap metachars (eg. [[:digit:]]) !!

-type (f|d)                       # [f]iles, [d]irectories
-L <path> -xtype l						    # find symlinks
-L <path> -type l				          # find broken symlinks
-maxdepth <value>							    # max number of levels to descend; `1` causes not to descend into directories
-mindepth <value>							    # opposite of `maxdepth`; useful when the root path (eg. `.`) must not be displayed (use `1`)
-follow                           # [follow] symlinks

-perm                                    # parameters can be either in number or literal ("u+w,g+w","o=w") format
      <value>                            # find where permissions are exactly <value>
      -<value>                           # find where all <value> permission are set
      /<value>                           # find where any of <value> permissions are set
-user <user>                      # [user]

-(a|c|m)[-|+](time|min)           # Accessed|Changed|Modified; time = n*hours, min = n*mins; +/- = more/less than
--mtime +7                        # mtime = file data last modified N*24 hours ago; +7 = modified more than 7 days ago
--mtime 0                         # 0 = created today
--mmin -<minutes>					        # mmin = modified in the last <minutes> minutes (don't forget the minus)

-size [+|-]<value>[c|k|M]         # bigger or greater than <value>; c/k/M = bytes/kilo/mega
-empty								            # empty files/directories

-print                            # when passing the content to the pipe, adds the filename to the start of every line
-print0 | xargs -0                # use null-char to separate filenames (useful for filenames with spaces) 
-print -quit							        # print one entry and quit; useful to check if there is one or more files in a directory (or matching)

-exec <command> {} \;		          # execute command for every file found, passing the name as {}; multiple -exec can be passed
-exec <command> {} +		          # execute command with as many files as possible (see https://unix.stackexchange.com/a/389706)
```

### Examples

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
find . -name 1.txt -not -path ./skipdir/* -not -path ./skipdir2/*
```

#### Search text inside multiple PDFs

```sh
find . -name '*.pdf' -exec sh -c 'pdftotext "{}" - | grep -i --with-filename --label="{}" --color "<pattern>"' \;
```

## Processes

### Parallel execution

#### Using GNU Parallle

The number of jobs is automatically limited to the number of cores.  
Quoting is not required.

```sh
ls -1 *.tar.* | parallel tar xvf		  # if unspecified, the argument is automatically appended
ls -1 | parallel zip -m {1}.zip {1}		# parameterized; can also use `{}`
```

Using the `:::` commands separator:

```
parallel echo ::: A B                 # in parallel: (echo A) (echo B)
parallel echo vmsav{1} ::: A B        # in parallel: (echo vmsavA) (echo vmsavB)
```

Examples:

```sh
# multiple commands for each job
ls -1 | parallel "avconv -i {} {}.wav && neroAacEnc -q 0.5 -if {}.wav -of {}.m4a"

# same command, 4 times
parallel 'ruby -e "while true; end"' ::: $(seq 4)
```

Specifying the input delimiter; WARNING! Newline is still considered a delimiter:

```sh
echo 'a b c' | parallel --delimiter ' ' echo
```

#### Using xargs

Differently from GNU Parallel, the command must can't be quoted.

- `-P <processes>`: use 0 for unlimited; based on the manpage, `-n` or `-L` should be used, but they weren't required with the personal use cases.

```sh
ls -1 *.txt | perl -pe 's/\.txt//' | xargs -P 4 -I {} mysql -e 'LOAD DATA INFILE "{}.txt" INTO TABLE "{}"'

# Prints the files, surrounded by double quotes.
# Double quotes needs to be escaped! Additionally, must escape the backslash, otherwise it's interpreted by Perl.
#
ls -1 *.txt | perl -pe 's/(.*)/\\"$1\\"/' | xargs -P 0 -I {} echo {}
```

Ignore failing commands:

```sh
seq 4 | xargs -I {} -P 0 sh -c 'aws ec2 delete-snapshot --snapshot-id {} || true'
```

## Dates

General format: `date +<format>`

Interpret a date: `date -d <input>`

### Formatting

Sequences:

- `%A`: full day of the week
- `%B`: full month
- `%H:%M:%s`
- `%Y-%m-%d`
- `%F %R` == `%Y-%m-%d %H:%M`

Operations:

```sh
# Print in a specific language:
#
LC_ALL=en_GB date --date='-1 month' +%B | tr '[:upper:]' '[:lower:]'
```

### Operations

Subtract (result is in seconds):

```sh
data_start_secs=`date -d "$data_start" +"%s"`
highlight_start_secs=`date -d "$highlight_start" +"%s"`

rel_highlight_start=`expr $highlight_start_secs - $data_start_secs`
```
