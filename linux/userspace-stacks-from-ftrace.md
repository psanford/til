# Userspace stacks from ftrace (golang)

Lets say you want to see the userspace stacks of process when it
makes syscall and you don't have {bpftrace,perf,gdb} available. With
a little bit of effort you can get this information out of ftrace!

I did this specifically for a Go (golang) process but it should work
for other runtimes. The only difference may be how you resolve symbol
names from addresses. That can be annoyingly tricky for JIT runtimes.

I also would expect this to work without framepointers.

It would be really cool if you could do stack sampling at a frequency
instead of by event but it doesn't seem like you can do that with ftrace.

### Capture a trace from ftrace


```bash

tracing=/sys/kernel/tracing

# Enable syscall events to collect userspace stacks from
echo 1 > $tracing/events/raw_syscalls/enable

# enable the function tracer
echo function > $tracing/current_tracer

# enable the userstacktrace option
echo userstacktrace > $tracing/trace_options

# Use absolute addresses so we can match things up with the symbol dump
# ASLR will probably mess this up
echo nosym-userobj > $tracing/trace_options

# enable tracing of child pids along with parent
echo event-fork > $tracing/trace_options

# only trace functions in the pid we care about
# if you have a running process with multiple threads
# use `ps -T -p SOME_PROCESS_PID -o tid= > $tracing/set_ftrace_pid
echo SOME_PROCESS_PID > $tracing/set_ftrace_pid

# start tracing
echo 1 > $tracing/tracing_on
# wait for 5 seconds for events
sleep 5
# stop tracing
echo 0 > $tracing/tracing_on

# save off the trace
cat $tracing/trace > /tmp/trace.raw

# save off the symbols from the process binary
objdump -Tt /path/to/running/executable > /tmp/symbols

```

Now you should have a raw capture from ftrace and the symbols from objdump.
The ftrace capture has both kernel and userspace frames, but the userspace
frames are just addresses without the function name. We can lookup the function
name using the symbols file we saved off.

Here's a little ruby script that you can give the trace and symbol files to
and it will replace the raw addresses with function names:

```ruby
#/usr/bin/env ruby

if ARGV.length < 2
  STDERR.puts("usage: #{$0} <trace_file> <symbol_file>")
  exit 1
end


trace_file = ARGV[0]
symbol_file = ARGV[1]

symbols = []

File.open(symbol_file) do |f|
  f.each_line do |l|
    l.strip!

    fields = l.split(/\s+/)
    # looking for lines like:
    # 000000000042d620 g     F .text	000000000000012b              runtime.Stack
    if fields.length < 6
      next
    end

    if fields[3] != '.text'
      next
    end


    # append addr and function_name
    symbols << [fields[0].to_i(16), fields[5]]
  end
end

File.open(trace_file) do |f|
  f.each_line do |l|

    if m = l.match(/^\s+=>\s+<([0-9a-f]+)>\s*$/)
      addr = m[1].to_i(16)
      symbolMatch = symbols.bsearch {|s| s[0] >= addr }
      if symbolMatch
        puts " =>  <#{symbolMatch[1]}>"
      else
        print l
      end
    else
      print l
    end
  end
end
```

The resulting output will look like this:
```bash
# tracer: function
#
# entries-in-buffer/entries-written: 45330/6194959   #P:1
#
#                              _-----=> irqs-off
#                             / _----=> need-resched
#                            | / _---=> hardirq/softirq
#                            || / _--=> preempt-depth
#                            ||| /     delay
#           TASK-PID   CPU#  ||||    TIMESTAMP  FUNCTION
#              | |       |   ||||       |         |
             foo-3444  [000] .... 123215.643607: __hrtimer_init <-hrtimer_nanosleep
             foo-3444  [000] .... 123215.643607: do_nanosleep <-hrtimer_nanosleep
...
 =>  <syscall.Syscall6>
 =>  <internal/poll.(*FD).WriteTo>
 =>  <net.(*netFD).writeTo>
 =>  <net.(*conn).Close>
 =>  <net/http.persistConnWriter.ReadFrom>
 =>  <bufio.(*Writer).Write>
 =>  <net/http.(*persistConn).wroteRequest>
 =>  <runtime.gcWriteBarrier>
             foo-3445  [000] .... 123215.644747: SyS_write <-do_syscall_64
             foo-3445  [000] .... 123215.644747: __fdget_pos <-SyS_write
             foo-3445  [000] .... 123215.644748: __fget_light <-__fdget_pos
             foo-3445  [000] .... 123215.644748: __fget <-__fget_light
             foo-3445  [000] .... 123215.644749: vfs_write <-SyS_write
             foo-3445  [000] .... 123215.644749: rw_verify_area <-vfs_write
             foo-3445  [000] .... 123215.644749: security_file_permission <-rw_verify_area
             foo-3445  [000] .... 123215.644750: apparmor_file_permission <-security_file_permission
             foo-3445  [000] .... 123215.644750: common_file_perm <-apparmor_file_permission
             foo-3445  [000] .... 123215.644751: aa_file_perm <-common_file_perm
             foo-3445  [000] .... 123215.644751: __vfs_write <-vfs_write
             foo-3445  [000] .... 123215.644751: new_sync_write <-__vfs_write
             foo-3445  [000] .... 123215.644752: sock_write_iter <-new_sync_write
             foo-3445  [000] .... 123215.644752: sock_sendmsg <-sock_write_iter
             foo-3445  [000] .... 123215.644752: security_socket_sendmsg <-sock_sendmsg
             foo-3445  [000] .... 123215.644753: apparmor_socket_sendmsg <-security_socket_sendmsg
             foo-3445  [000] .... 123215.644753: aa_sock_msg_perm <-apparmor_socket_sendmsg
             foo-3445  [000] .... 123215.644753: aa_sk_perm <-aa_sock_msg_perm
             foo-3445  [000] .... 123215.644754: inet_sendmsg <-sock_sendmsg
             foo-3445  [000] .... 123215.644754: tcp_sendmsg <-inet_sendmsg
             foo-3445  [000] .... 123215.644754: lock_sock_nested <-tcp_sendmsg
             foo-3445  [000] .... 123215.644754: _cond_resched <-lock_sock_nested
             foo-3445  [000] .... 123215.644755: rcu_all_qs <-_cond_resched
             foo-3445  [000] .... 123215.644755: _raw_spin_lock_bh <-lock_sock_nested
             foo-3445  [000] .... 123215.644755: __local_bh_enable_ip <-lock_sock_nested
             foo-3445  [000] .... 123215.644756: tcp_sendmsg_locked <-tcp_sendmsg
             foo-3445  [000] .... 123215.644756: tcp_rate_check_app_limited <-tcp_sendmsg_locked
             foo-3445  [000] .... 123215.644756: tcp_send_mss <-tcp_sendmsg_locked
             foo-3445  [000] .... 123215.644756: tcp_current_mss <-tcp_send_mss
             foo-3445  [000] .... 123215.644757: ipv4_mtu <-tcp_current_mss
             foo-3445  [000] .... 123215.644757: tcp_established_options <-tcp_current_mss
             foo-3445  [000] .... 123215.644758: tcp_v4_md5_lookup <-tcp_established_options
...
 =>  <syscall.Syscall6>
 =>  <internal/poll.(*FD).ReadFrom>
 =>  <net.(*netFD).readFrom>
 =>  <net.(*conn).Write>
 =>  <net/http.newBufioReader>
 =>  <bufio.(*Reader).Peek>
 =>  <bufio.(*Reader).ReadLine>
 =>  <bufio.(*Reader).WriteTo>
             foo-3443  [000] .... 123215.645108: SyS_read <-do_syscall_64
             foo-3443  [000] .... 123215.645108: __fdget_pos <-SyS_read
             foo-3443  [000] .... 123215.645109: __fget_light <-__fdget_pos
             foo-3443  [000] .... 123215.645109: __fget <-__fget_light
             foo-3443  [000] .... 123215.645109: vfs_read <-SyS_read
             foo-3443  [000] .... 123215.645110: rw_verify_area <-vfs_read
             foo-3443  [000] .... 123215.645110: security_file_permission <-rw_verify_area
             foo-3443  [000] .... 123215.645110: apparmor_file_permission <-security_file_permission
             foo-3443  [000] .... 123215.645111: common_file_perm <-apparmor_file_permission
             foo-3443  [000] .... 123215.645111: aa_file_perm <-common_file_perm
             foo-3443  [000] .... 123215.645112: __fsnotify_parent <-security_file_permission
             foo-3443  [000] .... 123215.645112: fsnotify <-security_file_permission
             foo-3443  [000] .... 123215.645112: __vfs_read <-vfs_read
             foo-3443  [000] .... 123215.645112: new_sync_read <-__vfs_read
             foo-3443  [000] .... 123215.645113: sock_read_iter <-new_sync_read
             foo-3443  [000] .... 123215.645113: sock_recvmsg <-sock_read_iter
             foo-3443  [000] .... 123215.645113: security_socket_recvmsg <-sock_recvmsg
             foo-3443  [000] .... 123215.645114: apparmor_socket_recvmsg <-security_socket_recvmsg
             foo-3443  [000] .... 123215.645114: aa_sock_msg_perm <-apparmor_socket_recvmsg
             foo-3443  [000] .... 123215.645114: aa_sk_perm <-aa_sock_msg_perm
```
