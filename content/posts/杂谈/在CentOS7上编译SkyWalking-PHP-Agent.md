需要安装的项目链接：<https://github.com/apache/skywalking-php>
项目文档：<https://skywalking.apache.org/docs/skywalking-php/next/en/setup/service-agent/php-agent/readme/>

由于SkyWalking PHP Agent需要libclang 9.0+，CentOS7上并没有，scl上也没有，于是需要自行编译。
但是编译libclang需要的gcc版本，CentOS7又不满足，因此需要编译个新的gcc。

首先安装gcc 9.3（这一步可以省略，直接用scl源里面的devtoolset就行，我这里编译是因为某些特殊原因）：

```shell
yum install gcc make gmp-devel mpfr-devel libmpc-devel texinfo flex

# 安装在/opt/gcc-9
mkdir /opt/gcc-9

wget https://github.com/gcc-mirror/gcc/archive/refs/tags/releases/gcc-9.3.0.zip
unzip gcc-9.3.0.zip
cd gcc-releases-gcc-9.3.0/
./contrib/download_prerequisites
cd ..
mkdir objdir
cd objdir
$PWD/../gcc-releases-gcc-9.3.0/configure --prefix=/opt/gcc-9 --enable-languages=c,c++ --disable-multilib
make -j
make install
```

然后安装clang13：

```shell
yum install cmake3

# 安装在/opt/clang
mkdir /opt/clang

wget https://github.com/llvm/llvm-project/releases/download/llvmorg-13.0.1/llvm-project-13.0.1.src.tar.xz
tar -xvf llvm-project-13.0.1.src.tar.xz
cd llvm-project-13.0.1.src
mkdir build
cd build/

export CC=/opt/gcc-9/bin/gcc 
export CXX=/opt/gcc-9/bin/g++ 
export LD_LIBRARY_PATH=/opt/gcc-9/lib64/lib

# 只编译clang
cmake3 -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/opt/clang -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS=clang ../llvm
make -j
make install

unset CC
unset CXX
unset LD_LIBRARY_PATH

# 这一步比较关键，不软链会找不到符号
cd /opt/clang/lib/
ln -s /opt/gcc-9/lib64/libstdc++.so.6
# 检查下符号有没有缺失
ldd /opt/clang/lib/libclang.so.13.0.1
```

连protoc的版本都不足，也得编：

```shell
wget https://github.com/protocolbuffers/protobuf/archive/refs/tags/v3.6.1.zip -O protobuf-3.6.1.zipo
unzip protobuf-3.6.1.zip
cd protobuf-3.6.1

# 安装在mkdir /opt/protobuf
mkdir /opt/protobuf

./autogen.sh
./configure --prefix=/opt/protobuf
make -j
make install
```

Rust的安装就省略了，照着官方文档编就是了。

然后到了终于到了编译SkyWalking PHP Agent：

```shell
wget https://github.com/apache/skywalking-php/archive/refs/tags/v0.6.0.zip -O skywalking-php-0.6.0.zip

unzip skywalking-php-0.6.0.zip
cd skywalking-php-0.6.0

export PROTOC=/opt/protobuf/bin/protoc
export LIBCLANG_PATH=/opt/clang/lib

# 不加这一行，会导致缺少<stdbool.h>之类的文件，具体目录位置可以在/usr搜下`stdbool.h`
export C_INCLUDE_PATH=/usr/lib/gcc/x86_64-redhat-linux/4.8.2/include

phpize
./configure
make -j
make install
```

终于安装完了。
