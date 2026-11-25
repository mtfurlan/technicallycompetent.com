---
title: Commandline Interfaces & Help/Man Pages
date: 2022-04-11
---

One of the first ways a user will interact with your program is via `$program
-h`, `$program --help` or `man $program`

It is very much worth putting effort into making a good impression by both
making the help easy to read and descriptive, and following convention as much
as possible for the actual options

One way this page is going to fall short is the examples are made up
hodgepodges of stuff copied from several places, and do not clearly articulate
a usecase, and not clearly describing a usecase is one of my bigger complaints
with a lot of programs so I need to work on these examples

## Argument Standardization
### Terminology
```
$program -a -c option_argument --verbose some_file
```
* argument: Every whitespace delimited thing
* `-a`: option or flag
* `-c`: option or flag
* `option_argument`: option argument for -c
* `--verbose`: long option
* `some_file`: non-option or positional argument


### Expectations
Basically follow [GNU Program Argument Syntax Conventions][GNU] and then [POSIX][POSIX].
I highly recommend reading both of those, they're short.

But to summarize:
* Allow options after non-options
  * [GNU][GNU]
  * [POSIX.1-2017 12.2][POSIX] guideline 9 suggests the opposite but whatever
* single hyphen (`-`) as an argument means the standard streams
  * `cat -` will echo stdin to stdout
* double hyphen (`--`) means end of options, everything following is a non-option argument
  *  `git checkout develop -- someFileOnDevelop`
  * [POSIX.1-2017 12.2][POSIX] guideline 10
  * [GNU][GNU]
* `-o foo` and `-ofoo` are equivalent
  * [POSIX.1-2017 12.1.2][POSIX] recommends supporting this but not encouraging it
  * [GNU][GNU] says it's fine
* `-abc` is equivalent to `-a -b -c` as long as abc don't take arguments
  * [POSIX.1-2017 12.2][POSIX] guideline 5
  * [GNU][GNU]
* `-abco foo` is equivalent to `-a -b -c -o foo`
  * [POSIX.1-2017 12.2][POSIX] guideline 5 and 14
* Short options should not have optional arguments, long options may, using the = syntax
  * [POSIX.1-2017 12.2][POSIX] guideline 7
  * [GNU][GNU]
* Try to use common options so knowledge of other tools can kinda apply
  * [ESR's list of short options](http://www.catb.org/~esr/writings/taoup/html/ch10s05.html) (sorry but he's an annoyingly good source)
  * [common GNU long options](https://www.gnu.org/prep/standards/standards.html#Option-Table)

[GNU]: https://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html
[POSIX]: https://pubs.opengroup.org/onlinepubs/9699919799.2018edition/basedefs/V1_chap12.html

## Help Format

### Example of the format I want
based primarily off `dmesg`, `lsblk`, `ag`, and `jq`
```
Usage: ssh-setup [OPTIONS] <KEY> <USER> <HOSTNAME>
       ssh-setup [OPTIONS] <USER>@<HOSTNAME>
       ssh-setup [OPTIONS] <HOSTNAME>
  make an ssh key named KEY for USER and try to install it on HOSTNAME

  basically runs
  "ssh-keygen -t ed25519 -N '' -f ~/.ssh/${KEY:-HOSTNAME}_ed25519"

Options:
  -c, --config    Try to add to ssh-config
      --dry-run   Print the commands instead of running them
  -h, --help      Display this help and exit
```

```
Usage: crontab-ish [-u <USER>] (-e | -l)
  edit or list a crontab

Options:
  -e, --edit        Edit the crontab
  -l, --list        List the crontab
  -u, --user=USER   Use USER's crontab
  -h, --help        Display this help and exit
```

### Description of the format I want
* Synopsis at the top, then description of the tool, then a blank line, then
  option documentation
* `[--optional whatever]` shows `--optional whatever` is optional
  * POSIX.1-2017 12.1.7
* `<PARAMETER NAME>` shows `PARAMETER NAME` is a single parameter
  * POSIX.1-2017 12.1.4
* `a | b` means a or b, not both; `[-4 | -6]` if optional, `(-e | -l)` if one
  is required
  * POSIX.1-2017 12.1.8 uses `|` for mutually exclusive arguments; the `( )`
    grouping is from git and docopt
* Descriptions don't need to be full sentences
* Capitalize each option description
* No periods after each option description
* Hard 80 char line length limit
* Option descriptions start at least two spaces after the longest option, in a
  column that's a multiple of the indent width
* Help and other common options go last
* Write the synopsis as `[OPTIONS] <ARG>`: uppercase sets arguments apart from
  the surrounding text; an argument is required when it has no `[]` around it;
  `[OPTIONS]` already implies repetition, so it gets no `...`
* `...` means repeated things, see [POSIX.1-2017 12.1.9][POSIX]
  * for repeated optionals, the `...` can go inside or outside `[]`, both are
    common and mean the same in practice
  * `[OPTION]...` is the coreutils style (`ls`, `grep`, `df`, `cp`), and
    matches POSIX's `[-g option_argument]...` form for options
  * `[OPTION...]` is what glibc's argp prints (`tar`, `file`), and matches
    POSIX's `[operand...]` form for operands
* Exit 0 for `--help` and print it to stdout; for usage errors print to stderr
  and exit 2 (bash builtins, Python argparse)



### Examples from the wild that may not be what I want
```
Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).
Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all                  do not ignore entries starting with .
      --group-directories-first
                             group directories before files;
                               can be augmented with a --sort option, but any
                               use of --sort=none (-U) disables grouping
      --time-style=TIME_STYLE  time/date format with -l; see TIME_STYLE below
  -1                         list one file per line.  Avoid '\n' with -q or -b
      --help     display this help and exit
      --version  output version information and exit

```
```
lsd 0.20.1
An ls command with a lot of pretty colors and some other stuff.

USAGE:
    lsd [FLAGS] [OPTIONS] [--] [FILE]...

FLAGS:
    -a, --all               Do not ignore entries starting with .
    -h, --human-readable    For ls compatibility purposes ONLY, currently set by default

OPTIONS:
        --blocks <blocks>...            Specify the blocks that will b
```


```
Usage: gcc [options] file...
Options:
  -pass-exit-codes         Exit with highest error code from a phase.
  --help                   Display this information.
  --target-help            Display target specific command line options.
  --help={common|optimizers|params|target|warnings|[^]{joined|separate|undocumented}}[,...].
                           Display specific types of command line options.
  (Use '-v --help' to display command line options of sub-processes).
  --version                Display compiler version information.
```

```
GNU bash, version 5.1.4(1)-release-(x86_64-pc-linux-gnu)
Usage:  bash [GNU long option] [option] ...
        bash [GNU long option] [option] script-file ...
GNU long options:
        --debug
        --version
Shell options:
        -ilrsD or -c command or -O shopt_option         (invocation only)
        -abefhkmnptuvxBCHP or -o option
```

```
jq - commandline JSON processor [version 1.6]

Usage:  jq [options] <jq filter> [file...]
        jq [options] --args <jq filter> [strings...]
        jq [options] --jsonargs <jq filter> [JSON_TEXTS...]

jq is a tool for processing JSON inputs, applying the given filter to
its JSON text inputs and producing the filter's results as JSON on
standard output.

The simplest filter is ., which copies jq's input to its output
unmodified (except for formatting, but note that IEEE754 is used
for number representation internally, with all that that implies).

For more advanced filters see the jq(1) manpage ("man jq")
and/or https://stedolan.github.io/jq

Example:

        $ echo '{"foo": 0}' | jq .
        {
                "foo": 0
        }

Some of the options include:
  -c               compact instead of pretty-printed output;
  --slurpfile a f  set variable $a to an array of JSON texts read from <f>;


```

```
Usage: tar [OPTION...] [FILE]...
GNU 'tar' saves many files together into a single tape or disk archive, and can
restore individual files from the archive.

Examples:
  tar -cf archive.tar foo bar  # Create archive.tar from files foo and bar.
  tar -tvf archive.tar         # List all files in archive.tar verbosely.
  tar -xf archive.tar          # Extract all files from archive.tar.

 Main operation mode:
  -A, --catenate, --concatenate   append tar files to an archive
  -c, --create               create a new archive
```

```
Usage:
  org.inkscape.Inkscape [OPTION…] file1 [file2 [fileN]]

Process (or open) one or more files.

Help Options:
  -?, --help                                 Show help options
```

```
Usage: perl [switches] [--] [programfile] [arguments]
  -0[octal]         specify record separator (\0, if no argument)

```

```
Nmap 7.80 ( https://nmap.org )
Usage: nmap [Scan Type(s)] [Options] {target specification}
TARGET SPECIFICATION:
  Can pass hostnames, IP addresses, networks, etc.
  Ex: scanme.nmap.org, microsoft.com/24, 192.168.0.1; 10.0.0-255.1-254
  -iL <inputfilename>: Input from list of hosts/networks
  -iR <num hosts>: Choose random targets
  --exclude <host1[,host2][,host3],...>: Exclude hosts/networks
  --excludefile <exclude_file>: Exclude list from file
HOST DISCOVERY:
  -sL: List Scan - simply list targets to scan
  -sn: Ping Scan - disable port scan
```

```
cd: cd [-L|[-P [-e]] [-@]] [dir]
    Change the shell working directory.

    Change the current directory to DIR.  The default DIR is the value of the
    HOME shell variable.
...
    Options:
      -L        force symbolic links to be followed: resolve symbolic
                links in DIR after processing instances of `..'
      -P        use the physical directory structure without following
                symbolic links: resolve symbolic links in DIR before
                processing instances of `..'

```

```
Usage: ag [FILE-TYPE] [OPTIONS] PATTERN [PATH]

  Recursively search for PATTERN in PATH.
  Like grep or ack, but faster.

Example:
  ag -i foo /bar/

Output Options:
     --ackmate            Print results in AckMate-parseable format
  -A --after [LINES]      Print lines after match (Default: 2)

```

```
Usage: grep [OPTION]... PATTERNS [FILE]...
Search for PATTERNS in each FILE.
Example: grep -i 'hello world' menu.h main.c
PATTERNS can contain multiple patterns separated by newlines.

Pattern selection and interpretation:
  -E, --extended-regexp     PATTERNS are extended regular expressions
  -Z, --null                print 0 byte after FILE name
```

```
Usage: sed [OPTION]... {script-only-if-no-other-script} [input-file]...

  -n, --quiet, --silent
                 suppress automatic printing of pattern space

```

```
Usage:
  mc [OPTION…] [this_dir] [other_panel_dir]


GNU Midnight Commander 4.8.26


Help Options:
  -h, --help                Show help options
  <snip>

Application Options:
  -V, --version             Displays the current version

```

```
Usage: df [OPTION]... [FILE]...
Show information about the file system on which each FILE resides,
or all file systems by default.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all             include pseudo, duplicate, inaccessible file systems
```

```
Usage:
 lsblk [options] [<device> ...]

List information about block devices.

Options:
 -D, --discard        print discard capabilities
```

```
Usage:
 dmesg [options]

Display or control the kernel ring buffer.

Options:
 -C, --clear                 clear the kernel ring buffer

```

```
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface]
           [-b bind_address] [-c cipher_spec] [-D [bind_address:]port]
           [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11]
           [-i identity_file] [-J [user@]host[:port]] [-L address]
           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]
           [-Q query_option] [-R address] [-S ctl_path] [-W host:port]
           [-w local_tun[:remote_tun]] destination [command]
```

```
Usage: file [OPTION...] [FILE...]
Determine type of FILEs.

      --help                 display this help and exit
  -v, --version              output version information and exit

```

```
usage:  crontab [-u user] file
        crontab [ -u user ] [ -i ] { -e | -l | -r }
                (default operation is replace, per 1003.2)
        -e      (edit user's crontab)
        -l      (list user's crontab)
        -r      (delete user's crontab)
        -i      (prompt before deleting user's crontab)
```

```
usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]

These are common Git commands used in various situations:

start a working area (see also: git help tutorial)
   clone             Clone a repository into a new directory
   init              Create an empty Git repository or reinitialize an existing one
```

```
VIM - Vi IMproved 8.2 (2019 Dec 12, compiled Oct 01 2021 01:51:08)

Usage: vim [arguments] [file ..]       edit specified file(s)
   or: vim [arguments] -               read text from stdin
   or: vim [arguments] -t tag          edit file where tag is defined
   or: vim [arguments] -q [errorfile]  edit file with first error

Arguments:
   --                   Only file names after this
   -h  or  --help       Print Help (this message) and exit
```


## Example Implementations
### Bash
Needs the GNU `getopt`, the BSD/macOS `getopt` doesn't support long options
```bash
#!/usr/bin/env bash
set -euo pipefail
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"

h ()
{
    if [ $# != 0 ]; then
        exec >&2 # redirect everything to stderr
        echo "$*"
    fi

    cat <<EOF
Usage: $(basename "${BASH_SOURCE[0]}") [OPTIONS] <BRANCH>
  BRANCH: name of branch to checkout

Options:
  -d, --dir=DIR       Do something with DIR (default: script dir)
  -f, --force         Force whatever
  -v, --verbose[=N]   Increase verbosity, or set it to N
  -q, --quiet         Decrease verbosity
  -h, --help          Display this help and exit
EOF

    exit $(( $# != 0 ? 2 : 0 )) # 2: usage error
}

msg() { >&2 printf '%s\n' "$*"; }
# shellcheck disable=SC2317 # example: overridden by the debugging die below
die() {
    [[ -n "${1-}" ]] && msg "${1-}"
    exit "${2:-1}"
}

# use if die is more of a debugging tool
die() {
    msg "[${FUNCNAME[1]}:${BASH_LINENO[0]}] ${1-}"
    exit "${2:-1}"
}

verbosity=0
logLevelGE()
{
    (( ${verbosity:-0} >= ${1:?must pass log level} ))
}

log()
{
    if logLevelGE "${1:?must pass log level}"; then
        shift
        msg "$@"
    fi
}


doStuff()
{
    local targetBranch=$1
    local dir=$2
    local force=$3

    if [ "$force" = true ]; then
        echo "force was true or whatever"
    fi
}


if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    if ! PARSED_ARGS=$(getopt -o d:fvqh --long dir:,force,verbose::,quiet,help -n "$0" -- "$@") ; then
        h "getopt issue ^"
    fi
    eval set -- "$PARSED_ARGS"

    dir="$DIR" # default to the script's directory
    force=false
    while true ; do
        case "$1" in
            -d|--dir) dir=$2; shift 2 ;;
            -f|--force) force=true; shift ;;
            -v) verbosity=$((verbosity+1)); shift ;;
            --verbose)
                if [[ -z "$2" ]]; then verbosity=$((verbosity+1))
                elif [[ "$2" =~ ^[0-9]+$ ]]; then verbosity=$((10#$2))
                else h "verbosity must be numeric, got '$2'"
                fi; shift 2 ;;
            -q|--quiet) verbosity=$(( verbosity > 0 ? verbosity - 1 : 0 )); shift ;;
            -h|--help) h ;;
            --) shift ; break ;;
            *) die "issue parsing args, unexpected argument '$1'!" ;;
        esac
    done

    targetBranch=${1:-}
    if [[ -z "$targetBranch" ]]; then
        h "need to pass in target branch"
    fi
    shift

    if (($# > 0)); then
        h "unexpected positional argument '$1'"
    fi

    doStuff "$targetBranch" "$dir" "$force"
fi
```

### C
```c
static void usage(FILE* out, char* name)
{
    fprintf(out, "Usage: %s [OPTIONS] <INFILE>\n", name);
    fprintf(out, "Print information about binary Testing Safety Message (.tsm) INFILE\n");
    fprintf(out, "\nOptions:\n");
    fprintf(out, "  -m, --message   Print every J2735 MessageFrame as it is processed\n");
    fprintf(out, "  -j, --json      Output JSON for the TSM header data\n");
    fprintf(out, "  -q, --quiet     Set verbosity to 0\n");
    fprintf(out, "  -h, --help      Display this help and exit\n");
    fprintf(out, "  -V, --version   Display version (%s) and exit\n", VERSION_COMMANDLINE_UTILS);
}

int main(int argc, char* argv[])
{
    bool print_message = false;
    bool json = false;
    int verbosity = 1;
    int opt;
    int option_index = 0;
    static struct option long_options[] =
    {
        { "message", no_argument, NULL, 'm' },
        { "json",    no_argument, NULL, 'j' },
        { "quiet",   no_argument, NULL, 'q' },
        { "help",    no_argument, NULL, 'h' },
        { "version", no_argument, NULL, 'V' },
        { NULL,      0,           NULL, 0   }
    };

    while ((opt = getopt_long(argc, argv, "mjqhV", long_options, &option_index)) != -1) {
        switch (opt) {
            case 'm':
                print_message = true;
                break;
            case 'j':
                json = true;
                break;
            case 'q':
                verbosity = 0;
                break;
            case 'h':
                usage(stdout, argv[0]);
                exit(0);
            case 'V':
                printf("%s version %s\n", argv[0], VERSION_COMMANDLINE_UTILS);
                exit(0);
            default: /* '?' */
                exit(2);
        }
    }

    // accept one arg
    if (argc - optind != 1) {
        fprintf(stderr, "ERROR: Missing or additional arguments found.\n");
        usage(stderr, argv[0]);
        return 2;
    }
    char* inputFilename = argv[optind];
    ...
    return 0;
}
```
