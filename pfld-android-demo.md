人脸检测: android手机上的PFLD模型Demo
PFLD android demo
arm face detect

2019.5.14


[TOC]

# 背景
   从参考1的内容来看，PFLD是简单、快速、超高精度人脸特征点检测算法.
   作者提供了  android apk demo 程序.

# 目的
   在android手机上，运行一下 PFLD 提供的 demo程序。
   然后，对比测试一下PFLD模型的速度.


# 测试1: PFLD 的 demo

   从参考1内，下载apk，安装到手机即可.
   手机用新手机，我是华为麦芒5/华为P10和华为畅享max.

   android手机测试过程： 
   - 安装apk后，启动PFLD程序.
   - 摄像头被启动，能看到人脸图像，无特征点信息输出。
   - 关闭程序，再次进入程序，发现有特征点输出。

# 测试2： 从源代码编译 PFLD demo   
   我有android stduio，直接打开源代码，编译并运行起来。
   作者提供的说明:
   ```
	This is the demo App for "PFLD: A Practical Facial Landmark Detector".

	There are an Android APK (PFLD.apk) and a folder (PFLD) including the source files. 
	Please notice that the whole project contains a face detector and our PFLD, 
	the model of PFLD is 2.1MB (.\PFLD\app\src\main\assets\FaceTracking\models\tracking.bin).



	How to use:
	--------------------------------------------------------------------------------------------------------
	For the APK:
	Step 1: Install it ;
	Step 2: Permit the App to read/write and visit *FRONT* camera
	Step 3: Have fun!
	--------------------------------------------------------------------------------------------------------
	************************************************************************************************
	--------------------------------------------------------------------------------------------------------
	For the source project:
	Step 1: Open the project with Android Studio;
	Step 2: Configure ndk.dir and sdk.dir in local.properties;
	Step 3: Build the App and install it;
	Step 4: Permit the App to read/write and visit *FRONT* camera
	Step 5: Have fun!
	--------------------------------------------------------------------------------------------------------
   ```

# PFLD 模型文件大小
   下面看看文件大小情况:
   
   |文件目录|大小(M)|说明
   |--|--|--
   |`PFLD\app\src\main\assets\FaceTracking\models`|4M|模型目录
   | 1.  tracking.bin|2.14M|the model of PFLD is 2.1MB|
   | 2. det1.bin/det1.param/det2.bin/det2.param/det3.bin/det3.param|1.89M|face detector模型|
   |`PFLD\app\src\main\jniLibs\armeabi-v7a\libTracking-lib.so`|3.15M|调用tracking.bin PFLD 模型的so|


# 参考链接:
## 参考1{#ref1} :
- [PFLD：简单、快速、超高精度人脸特征点检测算法](https://blog.csdn.net/dQCFKyQDXYm3F8rB0/article/details/88097078)
- [PFLD android demo apk](https://sites.google.com/view/xjguo/fld)
- [PFLD android demo apk（百度网盘提取码：glwr）](https://pan.baidu.com/s/16HjDy9TyotCVwDdd55oWVQ)
