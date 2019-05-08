# mac机器上编译tvm
[tvm on mac](https://github.com/zhaowd2001/tvm_phone/blob/master/tvm-mac-build.md)

2019/5/8
  目录
  
[TOC]

## 代码下载 

   
   
## 背景
   前端时间，在linux ubuntu 16.04 和 windows 上， 编译了tvm，也能跑起来。
   这几天想看看 tvm 在 iphone 手机上的运行速度，需要在mac下搭建编译环境。
   
## 目的
   本文描述如何在 mac 下，编译 tvm。

## cmake
   先要安装 cmake，目前最新版本是 3.14.

## 下载llvm mac 版本
   下载 llvm 6.0.1 版本, 解压到 ~/tvm/llvm-6.0.1

## 配置 config.cmake
   创建build目录，把`cmake/config.cmake`复制到build目录:
   
   ```
   mkdir build
   cp cmake/config.cmake build
   ```
   修改 build/config.cmake 内的配置:
   |修改内容|描述|原因|
   |LDFLAGS |`LDFLAGS 内添加 -lc++abi 添加进 `|否则XCode可能报错
   |USE_LLVM| `set(USE_LLVM <your path>/lvm-6.0.0/bin/llvm-config)`
   
## 编译 tvm
   ```
   cd build
   cmake ..
   make -j4
   ```

## 安装NNPACK[???]
   参见 参考6.
   ```
git clone --recursive https://github.com/Maratyszcza/NNPACK.git
cd NNPACK
# Add PIC option in CFLAG and CXXFLAG to build NNPACK shared library
sed -i "s|gnu99|gnu99 -fPIC|g" CMakeLists.txt
sed -i "s|gnu++11|gnu++11 -fPIC|g" CMakeLists.txt
mkdir build
cd build
# Generate ninja build rule and add shared library in configuration
cmake -G Ninja -D BUILD_SHARED_LIBS=ON ..
ninja
sudo ninja install

# Add NNPACK lib folder in your ldconfig
echo "/usr/local/lib" > /etc/ld.so.conf.d/nnpack.conf
sudo ldconfig
   ```

### 安装 PeachPy/confu
   ```
   pip install --user --upgrade git+https://github.com/Maratyszcza/PeachPy
   pip install --user --upgrade git+https://github.com/Maratyszcza/confu
   ```

   
### 安装 ninja
    参见 参考5.

### 编译 NNPACK
   ```
   confu setup
   python ./configure.py  
   ninja 
   ```

## 安装 python
### python 路径设置 
    把tvm 库的路径，加入python:
    ```
    export TVM_HOME=/path/to/tvm
    export PYTHONPATH=$TVM_HOME/python:$TVM_HOME/topi/python:$TVM_HOME/nnvm/python:${PYTHONPATH}
    ```
### 安装 anaconda tensorflow
   参见参考4.
   ```
   pip install --user tensorflow
   ```

### 安装 python 依赖模块
   `pip install --user numpy decorator attrs  tornado tornado psutil xgboost `
   
## 安装 opencv for python
   参见 参考7.
   ` pip install --user numpy wheel opencv-python `

   
### 测试 tvm python 是否正常

   ```
   python -c "import tvm; from tvm import relay"
   python -c "import tensorflow; import cv2; import numpy"
   ```

## 测试 tvm 导入 tensorflow 模型
   ```
   cd tutorials/frontend/
   python ./from_tensorflow.py
   ```


 
## 参考
  - 参考1: [tvm Install from Source](https://docs.tvm.ai/install/from_source.html#build-the-shared-library)
  - 参考2: [llvm download](http://releases.llvm.org/download.html)
  - 参考3： [Anaconda tensorflow for mac](https://www.anaconda.com/distribution/#macos)
  - 参考4： [在Mac下安装Tensorflow深度学习框架](https://blog.csdn.net/qq_35669389/article/details/80572717)
  - 参考5： [ninja mac](https://ninja-build.org/)
  - 参考6： [NNPACK nijia](https://docs.tvm.ai/install/nnpack.html)
  - 参考7: [python实现opencv学习一：安装、环境配置、工具](https://blog.csdn.net/u011321546/article/details/79499598)
  - 参考8： [NNPack 使用教程](https://blog.csdn.net/wjl_hdu/article/details/55190116)
  - 参考9: [NNPACK详解](https://blog.csdn.net/AMDS123/article/details/77284751)
