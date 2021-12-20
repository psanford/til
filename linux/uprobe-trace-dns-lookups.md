# Trace library calls and args via uprobes

Lets say you want to trace all calls to libc dns lookups (gethostbyname{2,},getaddrinfo).

If you have bpftrace or bcctools installed you can use https://github.com/iovisor/bpftrace/blob/master/tools/gethostlatency.bt. But lets say you don't have that available and want to do it with ftrace only.

First you need to figure out the address of the functions and where the hostname is set in the function call.

To get the address of the functions you can use objdump:

```
$ objdump -tT /lib/x86_64-linux-gnu/libc.so.6 | awk -v sym=gethostbyname  '$NF == sym && $4 == ".text"  { print $1; exit }'
00000000001336a0
$ objdump -tT /lib/x86_64-linux-gnu/libc.so.6 | awk -v sym=gethostbyname2  '$NF == sym && $4 == ".text"  { print $1; exit }'
0000000000133900
$ objdump -tT /lib/x86_64-linux-gnu/libc.so.6 | awk -v sym=getaddrinfo  '$NF == sym && $4 == ".text"  { print $1; exit }'
0000000000108fb0
```

The 'name' argument is the first argument for all of these functions. On x86-64, the pointer to the string will be in %rdi.

If you didn't know that, you can run a program under gdb and set a breakpoint in the function to figure this out:
```
/tmp$ gdb foo
(gdb) b gethostbyname
Breakpoint 1 at 0x10e0
(gdb) r
Starting program: /tmp/foo

Breakpoint 1, gethostbyname (name=0x555555556004 "google.com") at ../nss/getXXbyYY.c:94
(gdb) info args
name = 0x555555556004 "google.com"
(gdb) info frame
Stack level 0, frame at 0x7fffffffda40:
 rip = 0x7ffff7eef6a0 in gethostbyname (../nss/getXXbyYY.c:94); saved rip = 0x555555555260
 called by frame at 0x7fffffffdb10
 source language c.
 Arglist at 0x7fffffffda30, args: name=0x555555556004 "google.com"
 Locals at 0x7fffffffda30, Previous frame's sp is 0x7fffffffda40
 Saved registers:
  rip at 0x7fffffffda38
(gdb) info registers
rax            0x555555556004      93824992239620
rbx            0x5555555553a0      93824992236448
rcx            0x5555555553a0      93824992236448
rdx            0x30                48
rsi            0x0                 0
rdi            0x555555556004      93824992239620
rbp            0x7fffffffdb00      0x7fffffffdb00
rsp            0x7fffffffda38      0x7fffffffda38
r8             0x0                 0
r9             0x7ffff7fe0d50      140737354009936
r10            0x7                 7
r11            0x2                 2
r12            0x555555555100      93824992235776
r13            0x7fffffffdbf0      140737488346096
r14            0x0                 0
r15            0x0                 0
rip            0x7ffff7eef6a0      0x7ffff7eef6a0 <gethostbyname>
eflags         0x202               [ IF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0

```

If you are just tracing a single function use [Brendan Gregg's uprobe script](https://github.com/brendangregg/perf-tools/blob/master/user/uprobe):

```
$ sudo ./uprobe 'p:/lib/x86_64-linux-gnu/libc.so.6:getaddrinfo +0(%di):string'
Tracing uprobe getaddrinfo (p:getaddrinfo /lib/x86_64-linux-gnu/libc.so.6:0x108fb0 +0(%di):string). Ctrl-C to end.
            ping-452971  [001] d... 403218.519919: getaddrinfo: (0x7ff01745afb0) arg1="google.com"
            ping-452975  [000] d... 403225.878427: getaddrinfo: (0x7fb01bfaefb0) arg1="nytimes.com"
 ThreadPoolForeg-451888  [004] d... 403228.335369: getaddrinfo: (0x7fb683492fb0) arg1="www.google.com"
```

If you need to trace multiple functions at once you'll have to set this up yourself.

Using the locations of the functions from the objdump above add trace points to ftrace:

 echo '
> p:uprobes/getaddrinfo /lib/x86_64-linux-gnu/libc.so.6:0x0000000000108fb0 arg1=+0(%di):string
> p:uprobes/gethostbyname /lib/x86_64-linux-gnu/libc.so.6:0x00000000001336a0   arg1=+0(%di):string
> p:uprobes/gethostbyname2 /lib/x86_64-linux-gnu/libc.so.6:0x0000000000133900 arg1=+0(%di):string
> ' | sudo tee /sys/kernel/debug/tracing/uprobe_events

```

Then enable those probes:

```
$ echo 1 | sudo tee /sys/kernel/debug/tracing/events/uprobes/gethostbyname/enable
1
$ echo 1 | sudo tee /sys/kernel/debug/tracing/events/uprobes/getaddrinfo/enable
1
$ echo 1 | sudo tee /sys/kernel/debug/tracing/events/uprobes/gethostbyname2/enable
```

Watch events:

```
sudo cat /sys/kernel/debug/tracing/trace_pipe
```

## Cleanup

To cleanup:

```
$ echo 0 | sudo tee /sys/kernel/debug/tracing/events/uprobes/gethostbyname/enable
0
$ echo 0 | sudo tee /sys/kernel/debug/tracing/events/uprobes/getaddrinfo/enable
0
$ echo 0 | sudo tee /sys/kernel/debug/tracing/events/uprobes/gethostbyname2/enable


$ echo | sudo tee /sys/kernel/debug/tracing/uprobe_events

```
