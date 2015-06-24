# Introduction #

MemorySanitizer code itself can not be tested with MemorySanitizer for obvious reasons, but the rest of Clang/LLVM can. Instructions on this page show how to do it.

# Details #

MSan requires that all code in the process is instrumented (see [handling-external-code](http://clang.llvm.org/docs/MemorySanitizer.html#handling-external-code)). Luckily, the only external dependency of Clang is the C++ standard library (and, of course, libc, but MSan _almost_ takes care of it already).

The script referenced below is available from
[bootstrap.sh](https://code.google.com/p/memory-sanitizer/source/browse/bootstrap/bootstrap.sh).

Checkout the source:
```
svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm
cd llvm
R=$(svn info | grep Revision: | awk '{print $2}')
(cd tools && svn co -r $R http://llvm.org/svn/llvm-project/cfe/trunk clang)
(cd projects && svn co -r $R http://llvm.org/svn/llvm-project/compiler-rt/trunk compiler-rt)
(cd projects && svn co -r $R http://llvm.org/svn/llvm-project/libcxx/trunk libcxx)
(cd projects && svn co -r $R http://llvm.org/svn/llvm-project/libcxxabi/trunk libcxxabi)
```

Build Clang
```
mkdir build && cd build && cmake -GNinja -DCMAKE_BUILD_TYPE=Release .. && ninja
```

Run bootstrap.sh script that would first build libc++ and libc++abi with MSan instrumentation, then run the entire clang build with MSan and newly built libc++/libc++abi.
Build libcxx (and libcxxabi) with MemorySanitizer instrumentation:
```
CLANG=/path/to/clang ./bootstrap.sh
```

Note that building all targets in the instrumented tree will attempt to link newly built MSan runtime with MSan runtime from the previous stage, which is not a good idea.