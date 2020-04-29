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
