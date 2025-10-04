- [What are Exit Codes?](#what-are-exit-codes)
  - [What if the Exit Code Exceeds the Valid Range?](#what-if-the-exit-code-exceeds-the-valid-range)
  - [How do I Find the Current Exit Code?](#how-do-i-find-the-current-exit-code)
- [What Does Each Code Mean?](#what-does-each-code-mean)
  - [Standard](#standard)
  - [Custom](#custom)
  - [Signal-Related](#signal-related)
- [Sources](#sources)

# What are Exit Codes?
Exit codes are **unsigned 8-bit integers** returned by scripts, commands, and
programs to give more context to the nature of their failure condition (except exit code `0`) by
using pre-defined codes.

## What if the Exit Code Exceeds the Valid Range?
In more recent versions of Bash, the exit code is retained even if it exceeds 
the range of `0-255`. But, it is then "wrapped up". That Means code `256`
becomes code `0`, code `383` becomes `127`, and so on and so forth.

But, for compatibility's sake, it's best practice to just use codes within the range of `0-255`.

## How do I Find the Current Exit Code?
Bash stores exit codes in the special variable `?` as read-only. You can obtain this value by running:

```
$ ls --version
ls (GNU coreutils) 9.8
Copyright (C) 2025 Free Software Foundation, Inc.

$ echo "$?"
0
```

Get a non-zero exit status by running:

```
$ ls /non-existent
"/non-existent": No such file or directory

$ echo "$?"
2
```

# What Does Each Code Mean?
> [!NOTE]
> Do understand that not all the exit codes specified here are actually standardized, but mostly mere
> conventions.

Since there's literally 256 different codes, I won't get into too much detail with each code (nor
will I actually go over absolutely every code), but I still hope these tables help as a starting
point before researching on a specific exit code.

## Standard
"Standard" exit codes refers to exit codes specified in the
[POSIX](https://en.wikipedia.org/wiki/POSIX) standard, and the
[C Standard Library](https://en.wikipedia.org/wiki/C_standard_library).

| Exit Code |              Name              | Description                                                            |
|:---------:|:------------------------------:|:-----------------------------------------------------------------------|
|     0     |            Success             | The execution was a success.                                           |
|     1     |         General Error          | A generic error occurred during execution.                             |
|     2     |    Misuse of Shell Builtin     | Incorrect usage of a shell built-in command.                           |
|     3     |        Invalid Argument        | An incorrect or missing argument was provided to the script/command.   |
|     4     |      Initialization Error      | Insufficient memory/disk space, or invalid drive names/syntax.         |
|     5     |       Input/Output Error       | Failure to read or write data.                                         |
|     6     |   No Such Device or Address    | The specified device or address is unavailable.                        |
|    126    | Command Invoked Cannot Execute | Permission denied or the command is not executable.                    |
|    127    |       Command Not Found        | The command is not recognized or available in the environment's PATH   |
|    255    |    Exit Status Out of Range    | The program returned an exit status greater than 255.                  |

## Custom
"Custom" exit codes aren't universally recognized, however there are some common conventions derived
from software, like `sysexits.h`.

| Exit Code |           Name            | Description                                                                        |
|:---------:|:-------------------------:|:-----------------------------------------------------------------------------------|
|    64     | Command-Line Usage Error  | General syntax error in the command-line arguments.                                |
|    65     |     Data Format Error     | The input data format is incorrect/unexpected.                                     |
|    66     |     Cannot Open Input     | A specified file or input cannot be accessed.                                      |
|    67     |      Address Unknown      | Invalid or unknown destination address.                                            |
|    68     |     Hostname Unknown      | Unable to resolve the specified host.                                              |
|    69     |    Service Unavailable    | A service required to complete the task is unavailable.                            |
|    70     |  Internal Program Error   | An unhandled error occurred in the program logic.                                  |
|    71     |       System Error        | Generic system-related error, such as insufficient resources.                      |
|    72     | Critical OS File Missing  | Required system files are not accessible or missing.                               |
|    73     |       Cannot Create       | Failure to create file/directory.                                                  |
|    74     |         I/O Error         | Generic input/output failure.                                                      |
|    75     |     Temporary Failure     | A temporary condition caused the failure, such as a network issue.                 |
|    76     |   Remote protocol Error   | An error occurred in a remote communication protocol, such as SSH, SFTP, SCP, ect. |
|    77     |     Permission Denied     | The user or process lacks sufficient privileges.                                   |
|    78     |    Configuration Error    | An issue occurred in configuration files or settings.                              |

## Signal-Related
"Signal-related" codes refer to codes **the shell returns** when a process ends because it received a
UNIX signal rather than exiting normally. The convention is to add the sum of `128` and the signal
number.

|       Signal       | Signal Number (N) | Exit Code (128+N) | Description                                               |
|:------------------:|:-----------------:|:-----------------:|-----------------------------------------------------------|
|       SIGHUP       |         1         |        129        | Terminal hung up                                          |
|       SIGINT       |         2         |        130        | Interrupted by (`Ctrl+C`)                                 |
|      SIGQUIT       |         3         |        131        | Quit with core dump                                       |
|       SIGILL       |         4         |        132        | Illegal instruction                                       |
|      SIGTRAP       |         5         |        133        | Trace/breakpoint trap                                     |
|      SIGABRT       |         6         |        134        | Abort signal (`abort()`)                                  |
|       SIGBUS       |         7         |        135        | Bus error (bad memory access)                             |
|       SIGFPE       |         8         |        136        | Floating-point exception                                  |
|      SIGKILL       |         9         |        137        | Forcefully killed (`kill -9`)                             |
|      SIGUSR1       |        10         |        138        | User-defined signal `1`                                   |
|      SIGSEGV       |        11         |        139        | Segmentation fault                                        |
|      SIGUSR2       |        12         |        140        | User-defined signal `2`                                   |
|      SIGPIPE       |        13         |        141        | Broken pipe (e.g., head + closed stream)                  |
|      SIGALRM       |        14         |        142        | Timer expired                                             |
|      SIGTERM       |        15         |        143        | Graceful termination (kill default)                       |
|     SIGSTKFLT      |        16         |        144        | Stack fault (rare)                                        |
|      SIGCHLD       |        17         |        145        | Child process stopped/exited                              |
|      SIGCONT       |        18         |        146        | Continue if stopped                                       |
|      SIGSTOP       |        19         |        147        | Pause execution (cannot catch/ignore)                     |
|      SIGTSTP       |        20         |        148        | Terminal stop signal (`Ctrl+Z`)                           |
|      SIGTTIN       |        21         |        149        | Background process tried to read from terminal            |
|      SIGTTOU       |        22         |        150        | Background process tried to write to terminal             |
|       SIGURG       |        23         |        151        | Urgent data on socket (TCP out-of-band data) (rare)       |
|      SIGXCPU       |        24         |        152        | CPU time limit exceeded (e.g., `ulimit -t`)               |
|      SIGXFSZ       |        25         |        153        | File size limit exceeded (e.g., `ulimit -f`)              |
|     SIGVTALRM      |        26         |        154        | Virtual timer expired (e.g., `setitimer(ITIMER_VIRTUAL)`) |
|      SIGPROF       |        27         |        155        | Profiling timer expired (used by debugging tools)         |
|      SIGWINCH      |        28         |        156        | Window resize signal (e.g., terminal resized)             |
|       SIGIO        |        29         |        157        | I/O now possible (asynchronous I/O event)                 |
|       SIGPWR       |        30         |        158        | Power failure (triggered by UPS before shutdown) (rare)   |
|       SIGSYS       |        31         |        159        | Bad system call (invalid syscall)                         |
|  (None) – Unused   |        32         |        160        | No standard signal 32. Reserved/application-specific      |
|  (None) – Unused   |        33         |        161        | No standard signal 33. Rarely used                        |
|      SIGRTMIN      |        34         |        162        | First real-time signal (used for threading)               |
|     SIGRTMIN+n     |       35–?        |       163–?       | Real-time signals up to SIGRTMAX (varies by OS)           |
| SIGRTMAX (typical) |        63         |        191        | Last real-time signal (varies by OS; often 64)            |
| (Beyond SIGRTMAX)  |        64         |        192        | Custom/invalid on most systems                            |

# Sources
- [GNU.org - Exit Status](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)
- [FreeBSD.org - `sysexits.h`](https://man.freebsd.org/cgi/man.cgi?query=sysexits)
- [TLDP.org - Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/index.html)
- [ArchLinux.org - `sysexits.h`](https://man.archlinux.org/man/sysexits.h.3head.en)
