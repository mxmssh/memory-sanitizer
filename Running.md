Follow instructions in UsingGit under "Building". They will get you an MSan-enabled Clang compiler in llvm/build/bin/clang.

Compile and link your code with -fmemory-sanitizer. Run the resulting binary.

To get symbolized reports, define an environment variable like this:
```
MSAN_SYMBOLIZER_PATH=/path/to/llvm/build/projects/compiler-rt/utils/llvm-symbolizer/llvm-symbolizer
```

Reports can also be post-symbolized by feeding them into the following:
```
LLVM_SYMBOLIZER_PATH=/path/to/llvm/build/projects/compiler-rt/utils/llvm-symbolizer/llvm-symbolizer /path/to/llvm/projects/compiler-rt/lib/asan/scripts/asan_symbolize.py
```

The method above will only work reliably if you build the whole program with -fmemory-sanitizer, including all the libraries it depends on, with the exception of libc.