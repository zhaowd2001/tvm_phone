# tvm_phone
tvm arm gpu opencl

2019.4.27

# TVM:手机[gpu|cpu]上运行Tensorflow模型
[TOC]
## 目的
描述在手机上运行tvm的过程,分gpu/cpu两种情况.
手机上运行tvm的话，考虑到要支持android和ios两个平台，用c++来提高可移植性.

## 手机cpu运行TVM
用c++调用tvm，参考本文末尾的 [参考2](#ref2) 和 [参考3](#ref3) 就可以了.

## 手机gpu运行TVM
### 测试一下你的手机是否支持gpu
在手机上安装 [参考5(Opencl-Z)](#ref5)，能查看手机gpu信息。
华为mate7根本不支持gpu运算.
华为麦芒5看起来支持gpu运算，但tvm会运行出错.
华为Mate10/P10/小米6就可以运行tvm, 华为麦芒5/小米5就不行。

### 准备TVM C++代码
要在手机上运行gpu，需要参考本文末尾的[参考4](#ref4)才行.
[参考4](#ref4)和[参考2](#ref2)/[3](#ref3)的区别是：
`device_type从kDLCPU变成了kDLGPU.`

为了在手机上运行gpu，[参考4](#ref4)内的 kDLGPU，要修改成 kDLOpenCL;

如果直接操作了gpu数据，opencl会报告错误:
`OpenCL Error, code=-38: CL_INVALID_MEM_OBJECT`

[参考4](#ref4)和[参考2](#ref2)/[3](#ref3)还有一个重要的区别，就是对输入/输出数据的处理。
为了把数据复制进gpu,必须调用 `TVMArrayCopyFromBytes`。
为了把数据从gpu复制出来，必须调用 `TVMArrayCopyToBytes`。

### TVM device_type
Pc上使用nvida gpu的话，编程接口用cuda.
手机上使用gpu的话，编程接口需要用OpenCL库.
1. 在tvm内，要使用cpu的话，运算device_type是kDLCPU(1)。
2. 要使用手机opencl gpu的话，运算device_type是kDLOpenCL(4)。
3. 要使用pc nvida gpu的话，运算device_type是kDLGPU(2)

### 准备opencl.h和.so文件
编译tvm，需要opencl的头文件.
从[参考6](#ref6)下载opencl.h头文件.
手机上编译tvm，还需要 libOpenCL.so文件.
可以直接从你的手机内，找到libOpenCL.so，下载下来，放到你的ndk目录内去.
我的手机和pc上的ndk目录位置：  

|手机指令集|从手机上下载|复制到NDK目录|
|--|--|--|
|Armeabi-v7|/vendor/lib/libOpenCL.so|C:\Users\<your name>\AppData\Local\Android\Sdk\android-ndk-r19\ndk-bundle\toolchains\llvm\prebuilt\windows-x86_64\lib64\clang\8.0.2\lib\linux\arm|
|Arm64-v8|/vendor/lib64/libOpenCL.so|C:\Users\<your name>\AppData\Local\Android\Sdk\android-ndk-r19\ndk-bundle\toolchains\llvm\prebuilt\windows-x86_64\lib64\clang\8.0.2\lib\linux\aarch64|

### 准备手机UI
准备好以上的c++代码,opencl.h/.so,就可以编译出tvm so库文件了。
为了在手机上运行起来，需要java UI界面。
可以用tvm自带的android demo([参考7](#ref7))，编译后运行起来。
Tvm自带的代码会从网络下载数据，并且由于调用了linux命令，只能在linux下编译。

### 运行tvm
编译出android demo app后，就能在手机上测试了。


## 手机gpu运行速度测试
我们使用gpu的目的是为了提高速度，但是据说是opencl的一个bug，导致tvm gpu在手机上失去了速度优势。
. 问题1是：tvm第一次加载运行模型，opencl调用会非常慢，然后后面就会快起来。对手机app来讲，启动慢，是不可忍受的。

. 问题2是：tvm使用gpu的话，推理(run)确实很快，但是要取得推理(run)的结果（get_output），却非常慢。
问题2参见[参考8](#ref8)。

考虑到这两个问题，在手机上，还是用cpu吧，gpu还没法实用。

# 参考

## 名词解释:
. 名词1 TVM: 优化模型运算速度，生成能在手机上运行的二进制代码。
. 名词2 GPU：比cpu运算快，专门用于大数据运算.
. 名词3 Tensorflow: google推的人工智能模型训练框架.

## 参考链接:
### 参考1{#ref1}
tvm网站 https://tvm.ai/

### 参考2{#ref2}
在CPP下使用TVM来部署mxnet模型（以Insightface为例）
https://zhuanlan.zhihu.com/p/55996985?utm_source=wechat_timeline&utm_medium=social&utm_oi=882552529345466368&from=timeline

### 参考3{#ref3}
一步一步解读神经网络编译器TVM(二)利用TVM完成C++端的部署
https://zhuanlan.zhihu.com/p/60981432

### 参考4手机上运行tvm gpu c++{#ref4}
tvm_deploy_gpu_sample.cpp
https://gist.github.com/masahi/d6ad36890d087f866de185f19aac3814

### 参考5Opencl-Z app{#ref5}
检测 android是否支持 opencl(GPU)
https://stackoverflow.com/questions/26795921/does-android-support-opencl
You can use OpenCL-Z Android to check the available and capabilities of OpenCL on Android devices.
OpenCL-Z apk:
https://www.allfreeapk.com/opencl-z,1359401/download.html

### 参考6下载 opencl.h{#ref6}
https://stackoverflow.com/questions/29082524/android-studio-fatal-error-cl-cl-h-no-such-file-or-directory

### 参考7 Tvm的android demo(linux ){#ref7}
Apps/android_deploy
https://github.com/dmlc/tvm/tree/master/apps/android_deploy

### 参考8 How to make the GPU to CPU memory copy faster? #979{#ref8}
https://github.com/dmlc/tvm/issues/979?from=timeline

clEnqueueReadBuffer is too slow
https://stackoverflow.com/questions/32190761/clenqueuereadbuffer-is-too-slow?from=timeline

