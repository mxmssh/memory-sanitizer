# Getting the source #

See http://code.google.com/p/memory-sanitizer/source/checkout

# Updating the source #

git pull

# Making a change #

```
*fixfixfix*
git commit -a
git push
```

# Merging from upstream #

```
git checkout upstream  # this branch tracks upstream changes
rm -rf llvm
svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm
LLVM_REV=`svnversion llvm`
svn co -r $LLVM_REV http://llvm.org/svn/llvm-project/cfe/trunk llvm/tools/clang
svn co -r $LLVM_REV http://llvm.org/svn/llvm-project/compiler-rt/trunk llvm/projects/compiler-rt
svn co -r $LLVM_REV http://llvm.org/svn/llvm-project/libcxx/trunk llvm/projects/libcxx
find llvm -name ".gitignore" -delete
find llvm -name ".svn" -exec rm -rf {} ';'
git add llvm
git commit -a -m "Update to upstream r${LLVM_REV}."
git checkout master
git merge upstream
git push --all
```

# Diff from upstream #

```
# Full diff
git diff  upstream..master

# Stats
git diff --stat=100,80 upstream..master

# LLVM bits
git diff  upstream..master llvm/include llvm/lib llvm/test
```

# Building #
```
mkdir llvm/build
cd llvm/build
cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON ..
make -j25
cd ../projects/compiler-rt/lib/msan/
make t
make to
```
See msandr/README.txt for MSandr build instructions.