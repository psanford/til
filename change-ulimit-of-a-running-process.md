# Change the ulimit of a running process

Print the limits for a given pid:

```bash
$ prlimit --pid 3123
RESOURCE   DESCRIPTION                             SOFT      HARD UNITS
AS         address space limit                unlimited unlimited bytes
CORE       max core file size                         0 unlimited bytes
CPU        CPU time                           unlimited unlimited seconds
DATA       max data size                      unlimited unlimited bytes
FSIZE      max file size                      unlimited unlimited bytes
LOCKS      max number of file locks held      unlimited unlimited locks
MEMLOCK    max locked-in-memory address space  67108864  67108864 bytes
MSGQUEUE   max bytes in POSIX mqueues            819200    819200 bytes
NICE       max nice prio allowed to raise             0         0
NOFILE     max number of open files                1024   1048576 files
NPROC      max number of processes                62743     62743 processes
RSS        max resident set size              unlimited unlimited bytes
RTPRIO     max real-time priority                     0         0
RTTIME     timeout for real-time tasks        unlimited unlimited microsecs
SIGPENDING max number of pending signals          62743     62743 signals
STACK      max stack size                       8388608 unlimited bytes

```

Change the limits for core file size:
```bash
$ prlimit --core=unlimited:unlimited --pid 3123

$ prlimit --core  --pid 3123
RESOURCE DESCRIPTION             SOFT      HARD UNITS
CORE     max core file size unlimited unlimited bytes

```
