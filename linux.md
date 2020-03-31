# Linux

- [Linux](#linux)
  - [Processes](#processes)
    - [Parallel execution](#parallel-execution)
      - [Using GNU Parallle](#using-gnu-parallle)
      - [Using xargs](#using-xargs)

## Processes

### Parallel execution

#### Using GNU Parallle

The number of jobs is automatically limited to the number of cores.  
Quoting is not required.

```sh
ls -1 *.tar.* | parallel tar xvf		  # if unspecified, the argument is automatically appended
ls -1 | parallel zip -m {1}.zip {1}		# parameterized; can also use `{}`
```

Using the `:::` separator:

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
