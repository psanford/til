# ebpf notes


## Maps

Maps are memory areas that are accessible to your ebpf program and possibly also a userspace program.

There are a number of uses for maps. One common use case is for memory space that exceeds the (small) stack size
allowed for bpf programs. If you need more than 512 bytes, you'll need to use an ebpf map instead of a stack variable.

Map types:
- `BPF_MAP_TYPE_UNSPEC`:
- `BPF_MAP_TYPE_HASH`: map[long long][N]byte
- `BPF_MAP_TYPE_ARRAY`: [][N]byte
- `BPF_MAP_TYPE_PROG_ARRAY`: used for tail calls
- `BPF_MAP_TYPE_PERF_EVENT_ARRAY`: can be used for ring buffer (perf based)
- `BPF_MAP_TYPE_PERCPU_HASH`: same as HASH, but cpu local
- `BPF_MAP_TYPE_PERCPU_ARRAY`: same as ARRAY, but cpu local
- `BPF_MAP_TYPE_STACK_TRACE`: stack trace storage `bpf_get_stackid()`
- `BPF_MAP_TYPE_CGROUP_ARRAY`: used for checking skb membership in a cgroupv2
- `BPF_MAP_TYPE_LRU_HASH`: LRU hash
- `BPF_MAP_TYPE_LRU_PERCPU_HASH`: LRU hash by CPU
- `BPF_MAP_TYPE_LPM_TRIE`:  Longest Prefix Match TRIE
- `BPF_MAP_TYPE_RINGBUF`: Ring buffer (prefer over PERF_EVENT_ARRAY for newer kernels)


## Useful links

- [eBPF, part 2: Syscall and Map Types](https://www.ferrisellis.com/content/ebpf_syscall_and_maps/)
- [BPF Ring Buffer (kernl doc)](https://www.kernel.org/doc/html/latest/bpf/ringbuf.html)
- [BPF ring buffer](https://nakryiko.com/posts/bpf-ringbuf/)
- [BPF ring buffer examples](https://github.com/anakryiko/bpf-ringbuf-examples)
