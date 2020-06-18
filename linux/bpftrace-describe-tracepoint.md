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

You can work around that by manually declaring and casting the type
(https://github.com/iovisor/bpftrace/issues/999):
```
#include <linux/types.h>

struct data {
  long pad;
  int filename;
  int flags;
  int mode;
}

tracepoint:fs:do_sys_open {
  $data = (struct data*)args;
  printf("filename=%s\n", str((uint64)args + (uint64)($data->filename & 0xFFFF)));
}
```
