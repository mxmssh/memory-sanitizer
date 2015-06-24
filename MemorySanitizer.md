# Introduction #

MemorySanitizer (MSan) is a detector of uninitialized memory reads in C/C++ programs.

Uninitialized values occur when stack- or heap-allocated memory is read before it is written. MSan
detects cases where such values affect program execution.

MemorySanitizer is bit-exact: it can track uninitialized bits in a bitfield. It will tolerate
copying of uninitialized memory, and also simple logic and arithmetic operations with it. In general,
MemorySanitizer silently tracks the spread of uninitialized data in memory, and reports a warning
when a code branch is taken (or not taken) depending on an uninitialized value.

MemorySanitizer implements a subset of functionality found in Valgrind (Memcheck tool). It is significantly faster
than Memcheck (TODO:benchmark).

# Getting MemorySanitizer #

MemorySanitizer is part of LLVM trunk and 3.3 branch.

At this time, MemorySanitizer supports Linux x86\_64 only.


# Using MemorySanitizer #

To use MemorySanitizer, compile and link your program with -fsanitize=memory -fPIE -pie.

To get any stack traces, add -fno-omit-frame-pointer.

```
% cat umr.cc
#include <stdio.h>

int main(int argc, char** argv) {
  int* a = new int[10];
  a[5] = 0;
  if (a[argc])
    printf("xx\n");
  return 0;
}
%clang -fsanitize=memory -fPIE -pie -fno-omit-frame-pointer -g -O2 umr.cc
% ./a.out
==6726==  WARNING: MemorySanitizer: UMR (uninitialized-memory-read)
    #0 0x7fd1c2944171 in main umr.cc:6
    #1 0x7fd1c1d4676c in __libc_start_main /build/buildd/eglibc-2.15/csu/libc-start.c:226
```


## Origins tracking ##

MemorySanitizer can track back each uninitialized value to the memory allocation where it was created, and use this
information in reports. This behaviour is enabled with the `-fsanitize-memory-track-origins` flag.
It comes with additional 1.5x-2.5x slowdown, and makes the report from the previous example look like this:
```
%clang -fsanitize=memory -fsanitize-memory-track-origins -fPIE -pie -fno-omit-frame-pointer -g -O2 umr.cc
% ./a.out
==6726==  WARNING: MemorySanitizer: UMR (uninitialized-memory-read)
    #0 0x7fd1c2944171 in main umr.cc:6
    #1 0x7fd1c1d4676c in __libc_start_main /build/buildd/eglibc-2.15/csu/libc-start.c:226
  ORIGIN: heap allocation:
    #0 0x7f5872b6a31b in operator new[](unsigned long) msan_new_delete.cc:39
    #1 0x7f5872b62151 in main umr.cc:4
    #2 0x7f5871f6476c in __libc_start_main /build/buildd/eglibc-2.15/csu/libc-start.c:226
```


## Symbolization ##

Set MSAN\_SYMBOLIZER\_PATH environment variable to the path to llvm-symbolizer binary (normally built with LLVM), or just put llvm-symbolizer somewhere in your $PATH. MemorySanitizer will use it to symbolize reports on-the-fly.

## Using instrumented libraries ##

It is critical that you should build all the code in your program (including libraries it uses, in particular, C++ standard library)
with MemorySanitizer. See LibcxxHowTo for more details.

## Interface ##

```
#include <sanitizer/msan_interface.h>

// Dump shadow for a memory range. Shadow bit of 0 corresponds to initialized memory, 1 - to uninitialized memory.
__msan_print_shadow(ptr, size);

// Make memory range fully initialized. Does not change actual memory contents, but only MemorySanitizer perception of them.
__msan_unpoison(ptr, size);
```


## Debugging ##

```
set disable-randomization off
set overload-resolution off
br __msan_warning
br __msan_warning_noreturn
run
...
call __msan_print_shadow(&x, sizeof(x))
```

## MemorySanitizer algorithm ##

TODO: another wiki page