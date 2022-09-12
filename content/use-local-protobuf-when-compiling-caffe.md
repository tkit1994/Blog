---
title: 编译caffe时使用本地的protobuf
date: 2018-06-08 14:37:46
tags:
---

## 下载并编译protobuf

Protobuf的项目主页可以在[github](https://github.com/google/protobuf)上找到。在[Release页面](https://github.com/google/protobuf/releases)可以下载对应的代码。下载[protobuf-cpp-3.5.1.tar.gz](https://github.com/google/protobuf/releases/download/v3.5.1/protobuf-cpp-3.5.1.tar.gz)，别的版本应该也可以。

```bash
wget https://github.com/google/protobuf/releases/download/v3.5.1/protobuf-cpp-3.5.1.tar.gz
tar -xvf protobuf-cpp-3.5.1.tar.gz
cd protobuf-3.5.1
./configure --prefix="$HOME/.local"
make -j20
make install
```

## 编译caffe

```bash
unrar x caffe-sal.rar
cd caffe-sal
cp Makefile.config.example Makefile.config
```

编辑`Makefile.config`

修改

```Makefile
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib
```

为

```Makefile
INCLUDE_DIRS := $(PYTHON_INCLUDE) ${HOME}/.local/include /usr/local/include /usr/include/hdf5/serial/
LIBRARY_DIRS := $(PYTHON_LIB) ${HOME}/.local/lib /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu/hdf5/serial/
```

设置`PATH`环境变量，因为要使用刚才编译好的`protoc`,然后进行编译,j后面的数字是要使用的线程数，一般为cpu的线程数+1,只影响编译的速度

```bash
$ export PATH=${HOME}/.local/bin:$PATH
$ which protoc
/home/ncepu/.local/bin/protoc
$ make -j41
```

每次运行程序前设置环境变量`LD_LIBRARY_PATH`,或者把它加到.bashrc里面去。

```bash
export LD_LIBRARY_PATH=${HOME}/.local/lib:$LD_LIBRARY_PATH
```

## 查看链接库的版本

用下面的命令查看caffe链接的库的版本，可以看到是刚才编译好的版本。

```bash
ldd build/tools/caffe | grep protobuf
libprotobuf.so.15 => /home/ncepu/.local/lib/libprotobuf.so.15 (0x00007f7795fc1000)
```
