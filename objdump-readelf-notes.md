# objdump and readelf notes and examples

## List .note sections:

This is especially useful for seeing the `.build-id` section for matching to
external debug symbols in `/usr/lib/debug/.build-id/`

```
$ readelf -n /lib/x86_64-linux-gnu/libc-2.31.so

Displaying notes found in: .note.gnu.property
  Owner                Data size 	Description
  GNU                  0x00000010	NT_GNU_PROPERTY_TYPE_0
      Properties: x86 feature: IBT, SHSTK

Displaying notes found in: .note.gnu.build-id
  Owner                Data size 	Description
  GNU                  0x00000014	NT_GNU_BUILD_ID (unique build ID bitstring)
    Build ID: 099b9225bcb0d019d9d60884be583eb31bb5f44e

Displaying notes found in: .note.ABI-tag
  Owner                Data size 	Description
  GNU                  0x00000010	NT_GNU_ABI_TAG (ABI version tag)
    OS: Linux, ABI: 3.2.0

Displaying notes found in: .note.stapsdt
  Owner                Data size 	Description
  stapsdt              0x0000003a	NT_STAPSDT (SystemTap probe descriptors)
    Provider: libc
    Name: setjmp
    Location: 0x0000000000045d75, Base: 0x00000000001c1698, Semaphore: 0x0000000000000000
    Arguments: 8@%rdi -4@%esi 8@%rax
  stapsdt              0x0000003b	NT_STAPSDT (SystemTap probe descriptors)
    Provider: libc
    Name: longjmp
    Location: 0x0000000000045f21, Base: 0x00000000001c1698, Semaphore: 0x0000000000000000
    Arguments: 8@%rdi -4@%esi 8@%rdx
  stapsdt              0x00000042	NT_STAPSDT (SystemTap probe descriptors)
    Provider: libc
    Name: longjmp_target
    Location: 0x0000000000045f3d, Base: 0x00000000001c1698, Semaphore: 0x0000000000000000
    Arguments: 8@%rdi -4@%eax 8@%rdx
  stapsdt              0x0000003b	NT_STAPSDT (SystemTap probe descriptors)
    Provider: libc
    Name: longjmp
    Location: 0x0000000000045fb7, Base: 0x00000000001c1698, Semaphore: 0x0000000000000000
    Arguments: 8@%rdi -4@%esi 8@%rdx
  stapsdt              0x00000042	NT_STAPSDT (SystemTap probe descriptors)
    Provider: libc
    Name: longjmp_target
    Location: 0x0000000000045fd3, Base: 0x00000000001c1698, Semaphore: 0x0000000000000000
    Arguments: 8@%rdi -4@%eax 8@%rdx
  stapsdt              0x0000003a	NT_STAPSDT (SystemTap probe descriptors)
    Provider: libc
    Name: lll_lock_wait_private
    Location: 0x0000000000097736, Base: 0x00000000001c1698, Semaphore: 0x0000000000000000
    Arguments: 8@%rdi

    ...

  stapsdt              0x00000042	NT_STAPSDT (SystemTap probe descriptors)
    Provider: libc
    Name: longjmp_target
    Location: 0x0000000000132a6d, Base: 0x00000000001c1698, Semaphore: 0x0000000000000000
    Arguments: 8@%rdi -4@%eax 8@%rdx
```


## Read a section


Dump an elf section list `.gnu_debuglink`.


```
objdump -s -j .gnu_debuglink /lib/x86_64-linux-gnu/libc.so.6

/lib/x86_64-linux-gnu/libc.so.6:     file format elf64-x86-64

Contents of section .gnu_debuglink:
 0000 6c696263 2d322e33 312e736f 00000000  libc-2.31.so....
 0010 1cf080e3                             ....
```
