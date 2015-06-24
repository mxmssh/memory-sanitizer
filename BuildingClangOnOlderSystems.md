
```
sudo apt-get install --yes libgmp-dev  libmpfr-dev  libmpc-dev unzip zip
wget http://www.netgull.com/gcc/releases/gcc-4.8.2/gcc-4.8.2.tar.bz2
tar xf gcc-4.8.2.tar.bz2
cd gcc-4.8.2/
./configure --prefix=/usr/local/gcc-4.8.2
make -j20
sudo make install
```

```
cd /code/llvm
mkdir build0; cd build0
cmake -GNinja -DCMAKE_BUILD_TYPE=Release  -DLLVM_ENABLE_ASSERTIONS=ON -DCMAKE_C_COMPILER=/usr/local/gcc-4.8.2/bin/gcc -DCMAKE_CXX_COMPILER=/usr/local/gcc-4.8.2/bin/g++ -DGCC_INSTALL_PREFIX=/usr/local/gcc-4.8.2 ..
ninja clang
```

```
cd /code/llvm
mkdir build; cd build
cmake -GNinja -DCMAKE_BUILD_TYPE=Release  -DLLVM_ENABLE_ASSERTIONS=ON -DCMAKE_C_COMPILER=/code/llvm/build0/bin/clang -DCMAKE_CXX_COMPILER=/code/llvm/build0/bin/clang++ -DGCC_INSTALL_PREFIX=/usr/local/gcc-4.8.2 ..
ninja
ninja check-all
```

```
cd /code/llvm
mkdir build-configure; cd build-configure
CC=/code/llvm/build0/bin/clang CXX=/code/llvm/build0/bin/clang++ ../configure --enable-optimized --enable-assertions --with-gcc-toolchain=/usr/local/gcc-4.8.2
make -j35
```