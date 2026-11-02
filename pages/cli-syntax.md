---
title: Commandline Interfaces & Help/Man Pages
date: 2022-04-11
---

One of the first ways a user with interact with your progarm is via `$program
-h`, `$program --help` or `man $program`

It is very much worth putting effort into making a good impression by both
making the help easy to read and descripive, and following convention as much
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
* `-abco foo` is equivalent to `-abc -o foo`
  * [POSIX.1-2017 12.2][POSIX] guideline 5
* Short options should not have optional arguments, long options may using the = syntax
  * [POSIX.1-2017 12.2][POSIX] guideline 7
  * [GNU][GNU]
* TODO: In 2022 I said I ignore [POSIX.1-2017 12.2][POSIX] guideline 14 with "it's neat but I don't want to support it", but I can't work out what it means today
* Try to use common options so knowledge of other tools can kinda apply
  * [ESR's list of short options](http://www.catb.org/~esr/writings/taoup/html/ch10s05.html) (sorry but he's an annoyingly good source)
  * [common GNU long options](https://www.gnu.org/prep/standards/standards.html#Option-Table)

[GNU]: https://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html
[POSIX]: https://pubs.opengroup.org/onlinepubs/9699919799.2018edition/basedefs/V1_chap12.html

## Help Format

### Example of the format I want
based primarily off `dmesg`, `lsblk`, `ag`, `jq`, and `tar`
```
Usage: ssh-setup [OPTION]... <keyname> <username> <hostname>
       ssh-setup [OPTION]... <username@hostname>
       ssh-setup [OPTION]... <hostname>
  make a key and try to install it on hostname

  just runs 'ssh-keygen -t ed25519 -N '' -f $keyfile'
  keyfile is ${hostname}_ed25591

Available options:
  -c, --config    Try to add to ssh-config
  -h, --help      Display this help and exit
  -V, --version   Display version and exit
```

### Description of the format I want
* Synopsis at the top, then description of the tool, then option documentation
* `[optional arg]` shows `optional arg` is optional
  * POSIX.1-2017 12.2.7
* `<required arg>` shows `required arg` is required
  * POSIX.1-2017 12.2.4
* Capitalize each option description
* Hard 80 char line length limit
* Option descriptions start at least two spaces after the longest option, in a
  column that's a multiple of the indent width
* Always put help and version last
* No periods after each option description
*  show version in version or at top or at bottom?
* -V --version
* ... mean repeated things, see [POSIX.1-2017 12.1.9][POSIX]
  * for repeated optionals, the ... can go inside or outside [], user's choice
  * TODO: gnu tar 1.34: `Usage: tar [OPTION...] [FILE]...`, does using both mean anything?



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
```bash
#!/usr/bin/env bash
set -euo pipefail

#shellcheck disable=SC2120 # optional arg
h ()
{
    if [ $# != 0 ]; then
        exec >&2 # redirect everything to stderr
        echo "$*"
    fi

  cat <<EOF
Usage: $(basename "${BASH_SOURCE[0]}") [OPTION]... <branch>
  branch: name of branch to checkout
Available options:
  -d, --dir=DIR       Do something with DIR or whatever
  -f, --force         Force whatever
  -v, --verbose[=N]   Increase verbosity, or set it to N
  -q, --quiet         Set verbosity to 0
  -h, --help          Display this help and exit
EOF

    exit $(( $# != 0 ))
}

msg()
{
    printf '%s\n' "$*" >&2
}

verbosity=0
logLevelGE()
{
    local level=${1:?must pass log level}
    if [[ "$verbosity" -ge "$level" ]]; then
        return 0
    fi
    return 1
}
log()
{
    local level=${1:?must pass log level}
    shift;
    if logLevelGE "$level"; then
        msg "$@"
    fi
}

die()
{
    local msg=$1
    local code=${2-1} # default exit status 1
    msg "$msg"
    exit "$code"
}

doStuff()
{
    local targetBranch=$1
    local dir=$2
    local force=$3

    if [ "$force" = true ]; then
        ...
    fi
}

# run only if not sourced
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    if ! PARSED_ARGS=$(getopt -o d:fvqh --long dir:,force,verbose::,quiet,help -n "$0" -- "$@") ; then
        h "getopt issue ^"
    fi
    eval set -- "$PARSED_ARGS"

    dir=""
    force=false
    while true ; do
        case "$1" in
            -d|--dir) dir=$2; shift 2 ;;
            -f|--force) force=true; shift ;;
            -v) verbosity=$((verbosity+1)); shift ;;
            --verbose)
                if [[ -z "$2" ]]; then verbosity=$((verbosity+1))
                elif [[ "$2" =~ ^[0-9]+$ ]]; then verbosity=$2
                else h "verbosity must be numeric, got '$2'"
                fi; shift 2 ;;
            -q|--quiet) verbosity=0; shift ;;
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
        die "unexpected positional argument '$1'" 2
    fi

    doStuff "$targetBranch" "$dir" "$force"
fi
```

### C
```c
static void usage(char* name)
{
    printf("Usage: %s [OPTION...] <infile>\n", name);
    printf("Print information about binary Testing Safety Message (.tsm) FILE\n");
    printf("\nOptions:\n");
    printf("  -m, --message   Print every J2735 MessageFrame as it is processed\n");
    printf("  -j, --json      Output JSON for the TSM header data\n");
    printf("  -q, --quiet     Set verbosity to 0\n");
    printf("  -h, --help      Show this help\n");
    printf("  -V, --version   Show version (%s)\n", VERSION_COMMANDLINE_UTILS);
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
                usage(argv[0]);
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
        printf("ERROR: Missing or additional arguments found.\n");
        usage(argv[0]);
        return 1;
    }
    char* inputFilename = argv[optind];
    ...
    return 0;
}
```
