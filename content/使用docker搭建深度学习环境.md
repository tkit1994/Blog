---
title: 使用docker搭建深度学习环境
date: 2018-06-10 10:18:22
tags:
    - docker
    - 深度学习
    - 环境搭建
---

## Docker

Docker是一种虚拟化容器，可能把运行环境与操作系统隔离起来，使开发环境与操作系统分离，方便多个开发环境的管理。与虚拟机相比，它更加轻量，方便。

## 系统环境

- Ubuntu 16.04 LTS
- Nvidia GTX Titan Xp(其它Nvidia的显卡应该也可以)

## 安装Nvidia驱动（已经安装过的可以跳过）

可以参照这篇[文章](https://websiteforstudents.com/install-proprietary-nvidia-gpu-drivers-on-ubuntu-16-04-17-10-18-04/)

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install nvidia-387
```

## 安装Docker-ce和Nvidia-Docker

### 安装Docker-ce

docker-ce的安装可以参照这里的[官方文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/#uninstall-old-versions)

```bash
## 卸载Ubuntu官方源中的docker.io（如果之前安装了的话）
$ sudo apt-get remove docker docker-engine docker.io
## 更新apt package index.
$ sudo apt-get update
## 安装一些要用到了一些包
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
## 添加docker官方的GPG KEY
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
## 添加docker官方的源
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
## 更新apt package index.
$ sudo apt-get update
## 安装docker-ce
$ sudo apt-get install docker-ce
```

### 安装Nvidia-docker

在docker-ce中并不能使用GPU,要安装nvidia-docker才可以，nvidia-docker的安装可以参照其[官方文档](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0))

```bash
## 配置软件源
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
## 安装Nvidia-docker
$ sudo apt-get install nvidia-docker2
```

### 配置Docker

```bash
## 启动Docker服务
sudo service docker start
## 将当前用户加入docker group
sudo gpasswd -a $USER docker
```

### 优化Dockerhub连接(可选)

[Dockerhub](https://hub.docker.com/)是一个托管docker镜像的地方，就像[Github](https://github.com/)是一个托管代码的地方一样。上面有大量别人已经配置好的docker环境可以直接使用，十分方便。但是在国内访问十分缓慢。

可以参照这个[博客](https://blog.csdn.net/evandeng2009/article/details/53893789)使用阿里云容器hub对Dockerhub进行加速，阿里云容器hub的加速是免费的，实测在学校能达到大概1M/s，而直接连接Dockerhub的速度则惨不忍睹。

![阿里云容器hub](https://s2.loli.net/2022/09/12/T8F5iIbmlevS3zH.png)

复制上面的专属加速地层地址后编辑`/etc/docker//etc/docker/daemon.json`
在后面加入如下一行(注意json的格式，要写在`{}`里面，上一行结尾要有`,`)：

``` json
"registry-mirrosrs":["刚才复制的地址"]
```

![修改daemon.json](https://s2.loli.net/2022/09/12/zAybL5qoOTdHpGM.png)

至此Docker和Nvidia环境安装配置完成

## 使用docker

### 简单使用

一些常用的镜像可以直接在dockerhub上搜索，然后通过dcoker运行。比如下面的这个[Tensorflow](https://hub.docker.com/r/tensorflow/tensorflow/), 通过下面的这个命令即可开箱及用。

```bash
nvidia-docker run -it -p 8888:8888 tensorflow/tensorflow:latest-gpu
```

### 使用Nvidia提供的cuda&cudnn环境镜像

如果在电脑上安装多个版本的`cuda`、`cudnn`等库并且管理其版本是一件很麻烦的事情。有了docker之后可以使用Nvidia官方提供的docker镜像，在这些镜像中cuda和cudnn都是已经安装配置好了的。如果dockerhub上面没有我们想要的一些环境的话，可以在这个环境的基础上继续构建我们的环境。

以`8.0-cudnn5-devel-ubuntu16.04`为例：

```bash
nvidia-docker run --name test -it nvidia/cuda:8.0-cudnn5-devel-ubuntu16.04 bash
```

如下图片所示可以进入对应的环境，然后可以像正常使用Linux一样继续构建环境。

![8.0-cudnn5-devel-ubuntu16.04](https://s2.loli.net/2022/09/12/YqjedI5EOHlUK8L.png)

要退出环境的话，输入`exit`或者使用快捷键`ctrl=d`就可以了。

配置好环境退出后后使用可以使用`docker commit`将`container`保存成`image`从而使环境持久化，方便生成同样的环境。

```bash
$ docker commit test test:test
sha256:e9338a3b4aede076ab566c44b1127f5c1b18fd3dcb7cb3e02e19856f21db0c5b
```

可以通过下面的命令新建一个同样的环境

```bash
nvidia-docker run -it --name test01 test:test bash
```

如果只是想运行刚才的`container`的话，可以用`docker container start`命令。

更加详细的使用方法可以参照Docker的[官方文档](https://docs.docker.com/)和一些中文教程。

### 使用Dockerfile

Dockerfile可以用来自动化构建Docker Image.

#### 构建caffe-sal

参考caffe官方的[Dockerfile](https://github.com/BVLC/caffe/tree/master/docker)编写自己的Docerfile

目录结构

![目结构](https://s2.loli.net/2022/09/12/8SvG21YhMFlLdVs.png)

Dockerfile的内容

```Dockerfile
FROM nvidia/cuda

RUN apt-get update && apt-get install -y --no-install-recommends \
 cpio \
        build-essential \
        git \
        wget \
        numactl \
        vim \
        libopenblas-dev\
        screen \
        libmlx4-1 libmlx5-1 ibutils  rdmacm-utils libibverbs1 ibverbs-utils perftest infiniband-diags \
        openmpi-bin libopenmpi-dev \
        libboost-all-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        libhdf5-serial-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopencv-dev \
        libprotobuf-dev \
        libsnappy-dev \
        protobuf-compiler

ENV CAFFE_ROOT=/opt/caffe
WORKDIR $CAFFE_ROOT

ADD caffe-sal caffe-sal
WORKDIR $CAFFE_ROOT/caffe-sal
ADD Makefile.config Makefile.config

RUN make -j40
```

Makefile.config的内容

```Makefile
### Refer to http://caffe.berkeleyvision.org/installation.html
## Contributions simplifying and improving our build system are welcome!

## cuDNN acceleration switch (uncomment to build with cuDNN).
## USE_CUDNN := 1

## CPU-only switch (uncomment to build without GPU support).
## CPU_ONLY := 1

## To customize your choice of compiler, uncomment and set the following.
## N.B. the default for Linux is g++ and the default for OSX is clang++
## CUSTOM_CXX := g++

## CUDA directory contains bin/ and lib/ directories that we need.
CUDA_DIR := /usr/local/cuda
## On Ubuntu 14.04, if cuda tools are installed via
## "sudo apt-get install nvidia-cuda-toolkit" then use this instead:
## CUDA_DIR := /usr

## CUDA architecture setting: going with all of them.
## For CUDA < 6.0, comment the *_50 lines for compatibility.
CUDA_ARCH := -gencode arch=compute_30,code=sm_30 \
  -gencode arch=compute_35,code=sm_35 \
  -gencode arch=compute_50,code=sm_50 \
  -gencode arch=compute_50,code=compute_50

## BLAS choice:
## atlas for ATLAS (default)
## mkl for MKL
## open for OpenBlas
BLAS := open
## Custom (MKL/ATLAS/OpenBLAS) include and lib directories.
## Leave commented to accept the defaults for your choice of BLAS
## (which should work)!
## BLAS_INCLUDE := /path/to/your/blas
## BLAS_LIB := /path/to/your/blas

## Homebrew puts openblas in a directory that is not on the standard search path
## BLAS_INCLUDE := $(shell brew --prefix openblas)/include
## BLAS_LIB := $(shell brew --prefix openblas)/lib

## This is required only if you will compile the matlab interface.
## MATLAB directory should contain the mex binary in /bin.
## MATLAB_DIR := /usr/local
## MATLAB_DIR := /Applications/MATLAB_R2012b.app

## NOTE: this is required only if you will compile the python interface.
## We need to be able to find Python.h and numpy/arrayobject.h.
PYTHON_INCLUDE := /usr/include/python2.7 \
  /usr/lib/python2.7/dist-packages/numpy/core/include
## Anaconda Python distribution is quite popular. Include path:
## Verify anaconda location, sometimes it's in root.
## ANACONDA_HOME := $(HOME)/anaconda
## PYTHON_INCLUDE := $(ANACONDA_HOME)/include \
  # $(ANACONDA_HOME)/include/python2.7 \
  # $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include \

## We need to be able to find libpythonX.X.so or .dylib.
PYTHON_LIB := /usr/lib
## PYTHON_LIB := $(ANACONDA_HOME)/lib

## Homebrew installs numpy in a non standard path (keg only)
## PYTHON_INCLUDE += $(dir $(shell python -c 'import numpy.core; print(numpy.core.__file__)'))/include
## PYTHON_LIB += $(shell brew --prefix numpy)/lib

## Uncomment to support layers written in Python (will link against Python libs)
## WITH_PYTHON_LAYER := 1

## Whatever else you find you need goes here.
INCLUDE_DIRS := $(PYTHON_INCLUDE) ${HOME}/.local/include /usr/local/include /usr/include/hdf5/serial/
LIBRARY_DIRS := $(PYTHON_LIB) ${HOME}/.local/lib /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu/hdf5/serial/

## If Homebrew is installed at a non standard location (for example your home directory) and you use it for general dependencies
## INCLUDE_DIRS += $(shell brew --prefix)/include
## LIBRARY_DIRS += $(shell brew --prefix)/lib

## Uncomment to use `pkg-config` to specify OpenCV library paths.
## (Usually not necessary -- OpenCV libraries are normally installed in one of the above $LIBRARY_DIRS.)
## USE_PKG_CONFIG := 1

BUILD_DIR := build
DISTRIBUTE_DIR := distribute

## Uncomment for debugging. Does not work on OSX due to https://github.com/BVLC/caffe/issues/171
## DEBUG := 1

## The ID of the GPU that 'make runtest' will use to run unit tests.
TEST_GPUID := 0

## enable pretty build (comment to see full commands)
Q ?= @

```

```bash
## 运行构建命令
$ docker build --rm -t caffe-sal:latest .
```

构建结果：

![构建结果](https://s2.loli.net/2022/09/12/9pyLmWl6eDs4U87.png)

运行构建好的`Image`:

```bash
nvidia-docker run it caffe-sal:latest bash
```

运行结果

![运行结果](https://s2.loli.net/2022/09/12/Fb2TBnStfR9lUcq.png)
