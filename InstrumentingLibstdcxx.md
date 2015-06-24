# Introduction #

MemorySanitizer (without a dynamic component) requires that the entire program code including libraries, (except libc/libm/libpthread, to some extent), is instrumented.

Any non-trivial C++ program depends on the C++ runtime library. The choice here is between libstdc++ and libc++. The latter can be built with MemorySanitizer quite easily, but, due to minor incompatibilities between them, code that works with libstdc++ sometimes must be changed to work with libc++.

Instructions below let one build MSan-instrumented libstdc++. The resulting library, if using dynamic linking, can be substituted in run-time with an LD\_LIBRARY\_PATH setting. Compilation of the user code is unaffected.

# Details #

The following process works at the moment with the google branch of gcc 4.8 http://gcc.gnu.org/viewcvs/gcc/branches/google/gcc-4_8/.

```
CLANG=$LLVM_BIN/clang
CLANGXX=$LLVM_BIN/clang++
CLANG_HEADERS=$($CLANGXX -v -xc++ -c /dev/null |& grep -E 'lib/clang/[01-9.]+/include$' | head -1 | sed 's/ \+//')
# We need gcc's unwind.h. Either use the one for system-wide gcc installation if its fresh enough, or build gcc from the source on the side.
GCC_HEADERS=$(g++ -v -xc++ -c /dev/null |& grep -E '/[01-9.]+/include$' | head -1 | sed 's/ \+//')
MSAN_CFLAGS="-fsanitize=memory -isystem $CLANG_HEADERS -isystem $GCC_HEADERS -g -O2 -fno-omit-frame-pointer"
MSAN_LDFLAGS="-fsanitize=memory"
mkdir build && cd build
CC="$CLANG" CXX="$CLANGXX" CFLAGS="$MSAN_CFLAGS" CXXFLAGS="$MSAN_CFLAGS" LDFLAGS="$MSAN_LDFLAGS" \
  ../libstdc++-v3/configure --enable-multilib=no
make -j10
```

That's it. Instrumented library is in src/.libs, debug version in src/debug/.libs.