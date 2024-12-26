由于历史遗留问题，需要在Centos6.8这个过时的系统版本上编译grpc，总结一下几个遇到的问题。

## grpc的编译步骤

来源于 [Install SkyWalking PHP Agent](https://github.com/SkyAPM/SkyAPM-php-sdk/blob/93716ed8843c32765dc8a6b65c78ef77a8f3b8e3/docs/install.md)

```bash
# 省略了依赖安装

git clone --depth 1 -b v1.34.x https://github.com/grpc/grpc.git /var/local/git/grpc
cd /var/local/git/grpc
git submodule update --init --recursive

# protobuf
cd /var/local/git/grpc/third_party/protobuf
./autogen.sh
./configure
make -j$(nproc)
sudo make install
sudo ldconfig
make clean

# grpc
cd /var/local/git/grpc
mkdir -p cmake/build
cd cmake/build
cmake ../.. -DBUILD_SHARED_LIBS=ON -DgRPC_INSTALL=ON
make -j$(nproc)
sudo make install
make clean
sudo ldconfig
```

## 问题总结

1. 不能用自带的cmake，因为Centos6.8的cmake是2.x的版本，虽然源里面有个cmake3，但是版本太旧了，可以让grpc编译通过，但是最后会发现缺少了libgrpc.so、libgrpc++.so等动态链接库，这个才是最坑的，所以还是自行编译安装个cmake吧。

```bash
wget https://github.com/Kitware/CMake/releases/download/v3.20.1/cmake-3.20.1.tar.gz
tar zxvf cmake-3.20.1.tar.gz
cd cmake-3.20.1
./bootstrap
make -j
make install
cmake --version
```

2. GCC版本过低，我安装了GCC 9.3的版本，自行编译安装的，安装前需要先安装依赖，幸好，Centos6.8源自带的那几个依赖的版本恰好够得上。

```bash
yum install gmp-devel mpfr-devel libmpc-devel
```

然后参考<https://gcc.gnu.org/wiki/InstallingGCC>，安装：

```bash
wget https://github.com/gcc-mirror/gcc/archive/refs/tags/releases/gcc-9.3.0.zip
unzip gcc-9.3.0.zip
cd gcc-releases-gcc-9.3.0/
./contrib/download_prerequisites
cd ..
mkdir objdir
cd objdir
$PWD/../gcc-releases-gcc-9.3.0/configure --enable-languages=c,c++ --disable-multilib
make -j
make install
```

并且，stdlibc++的版本也不够新，那么在objdir目录下，安装：
```bash
cd x86_64-pc-linux-gnu/libstdc++-v3
make -j
make install
```

然后export一下环境变量：

```bash
export LD_LIBRARY_PATH="/usr/local/lib64:/usr/local/lib:/usr/lib64:/usr/lib:/lib64:/lib"
export LD_RUN_PATH=$LD_LIBRARY_PATH
```

3. 编译grpc的时候会遇到这个报错：

> /var/local/git/grpc/third_party/boringssl-with-bazel/linux-x86_64/crypto/chacha/chacha-x86_64.S:1591: Error: suffix or operands invalid for `vpxor'

原因是bintuils版本过低，一样编译安装一个更加新的。

```bash
wget https://ftp.gnu.org/gnu/binutils/binutils-2.36.tar.gz
unzip binutils-2.36.tar.gz
tar zxvf binutils-2.36.tar.gz
cd binutils-2.36
./configure
make -j
make install
```

4. 编译grpc的时候会遇到这个报错：

> undefined reference to `clock_gettime'

只要在cmake之前加上下面的环境变量即可

```bash
export LDFLAGS=-lrt
```
