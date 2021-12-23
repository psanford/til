# Notes from Steven Rostedt Tracing With ftrace

https://www.youtube.com/watch?v=mlxqpNvfvEQ

These are some things I learned from this talk.

## ftrace instances

You can create multiple ftrace instances by doing a mkdir in `/sys/kernel/tracing/instances`. You can `rmdir /sys/kernel/tracing/instances/foo` to remove that instance.

## get (kernel) struct offsets via gdb

To get kernel struct offsets you can use the following trick:

- install kernel debug symbols (ubuntu: `apt-get install linux-image-$(uname -r)-dbgsym`)
- run `gdb /usr/lib/debug/boot/vmlinux-$(uname -r)`
- cast 0 to a pointer to the struct and print the address of the field:

```
(gdb) p &(((struct task_struct *)0)->pid)
$1 = (pid_t *) 0x8c0

(gdb) p &(((struct task_struct *)0)->comm)
$2 = (char (*)[16]) 0xa78
```


## error log

`/sys/kernel/tracing/error_log` will tell you what went wrong if you get errors from the tracing vfs interface

## trace_marker

You can write events into `/sys/kernel/tracing/trace_marker` from userspace to have those events show up in the trace log.
