# bpftrace describe tracepoint

We all know you can list tracepoints with:
```
$ bpftrace -l
```

If you want to see the arguments for a tracepoint you can run:
```
$ bpftrace -v -l tracepoint:fs:do_sys_open
tracepoint:fs:do_sys_open
    __data_loc char[] filename;
    int flags;
    int mode;
```

Note that this doesn't work for kprobes. For those you
have to use arg0..argN.

You also can't inspect fields of type `__data_loc` right now:
https://github.com/iovisor/bpftrace/issues/385.
