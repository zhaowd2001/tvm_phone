# 用cmake生成ios framework库
[cmake ios framework](https://github.com/zhaowd2001/tvm_phone/blob/master/tvm-cmake-ios.md)

2019/5/8
  目录
  
  [TOC]

## 代码下载 

   [cmake-ios-demo 版本1: cmake ](https://zwd.3wfocus.com/svn/files/trunk/tp/tvm/apps/tvm_demo1)
   
   [cmake-ios-demo 版本2: ios-cmake ](https://zwd.3wfocus.com/svn/files/trunk/tp/tvm/apps/tvm_demo2)
   
   [cmake-ios-demo 版本3: opencv2.framework ](https://zwd.3wfocus.com/svn/files/trunk/tp/tvm/apps/tvm_demo3)
   
   
## 背景
   如果你开发了一套c++代码，要在android和iphone两种手机上运行，就要在分别编译android和iphone两个平台下的动态库。
   
   android开发目前是用 Android Studio(SDK/NDK), iphone开发是用XCode.
   
   当你增加或减少一个源代码文件后，要在Android Studio和XCode内分别添加这个源代码。
   
   随着时间的推移，源文件变来变去，你会烦不胜烦。
   
## 目的
   本文描述如何用cmake，在android和iphone两个平台下，生成各自的库文件，供app调用.

## 版本1: cmake
### 代码目录结构

   我准备的演示代码目录如下:
   
   |代码目录|描述|
   |--|--|
   |tvm_model| 能生成 android 和 iphone两个平台的库.
   |         | tvm_model.hpp(开放给ios app调用) 
   |         | tvm_model.cc(功能实现函数)
   |         | CMakeLists.txt : cmake 需要的项目文件
   |         | android 平台: cmake生成的文件是libtvm_model.so 
   |         | ios 平台: cmake生成的文件是 tvm_model.framework
   |tvm_demo_android|android app,  调用 libtvm_model.so 库内的函数.
   |tvm_demo_ios    | iphone app, 调用 tvm_model.framework 库内的函数.
   
### 检查cmake是否安装
   用以下命令，确认你是否安装了cmake：
   `cmake --version`
   
   在我的mac机器上，输出是：
   `cmake version 3.13.1`
   
### 制作CMakeLists.txt
   根据CMake官方文档[framework](https://cmake.org/cmake/help/v3.13/prop_tgt/FRAMEWORK.html
),[resource](https://cmake.org/cmake/help/v3.13/prop_tgt/RESOURCE.html)，制作的CMakeLists.txt内容如下:
   
   ```
cmake_minimum_required(VERSION 3.11)

Project(tvm_model)
add_library(tvm_model SHARED
            tvm_model.cc
            tvm_model.hpp
)

set(RESOURCE_FILES
  readme.md
  # appresourcedir/appres.txt
)

set_target_properties(tvm_model PROPERTIES
  FRAMEWORK TRUE
  FRAMEWORK_VERSION A
  MACOSX_FRAMEWORK_IDENTIFIER cn.tvm.tvm_model
  # MACOSX_FRAMEWORK_INFO_PLIST Info.plist
  # "current version" in semantic format in Mach-O binary file
  VERSION 1.0.0
  # "compatibility version" in semantic format in Mach-O binary file
  SOVERSION 1.0.0
  PUBLIC_HEADER tvm_model.hpp
  XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY "iPhone Developer"
  RESOURCE "${RESOURCE_FILES}"
)
   ```

### 编译出 tvm_model.framework

   用以下命令编译出库文件:
   
   ```
   cd ./tvm_demo1/src/tvm_model
   mkdir build
   cd    build
   cmake ..
   make
   ```
   
### Resources 没有被复制
   我们期望 readme.md 被复制到 tvm_model.framework的Resources目录下,但是很上面的make命令生成的tvm_model.framework，却没有包含 readme.md.
   
   参照[cmake 打包 ios sdk](https://github.com/Tencent/ncnn/wiki/cmake-%E6%89%93%E5%8C%85-ios-sdk?from=singlemessage), 我们可以把 readme.md, 复制到 tvm_model.framework 目录.

## 版本2: ios-cmake
   这次，我们用网上推荐的 [ios-cmake](https://github.com/leetal/ios-cmake) 来试一下。
   ios-cmake 需要[cmake-3.14](https://cmake.org/download/) 版本，要下载安装一下。
   仿照它的例子，制作编译的 CMakeLists.txt如下：
   ```

cmake_minimum_required(VERSION 3.14)

Project (tvm_model C CXX)

# Includes
include_directories(${tvm_model_SOURCE_DIR})

set(RESOURCE_FILES1
  readme.md
)


add_library(tvm_model SHARED
            tvm_model_c.cc
            tvm_model_c.hpp
            tvm_model.mm
            tvm_model.h
)
# Debug symbols set in XCode project
set_xcode_property(tvm_model GCC_GENERATE_DEBUGGING_SYMBOLS YES "All")

set(HEADER_FILES1
  tvm_model_c.hpp
  tvm_model.h
)

set(RESOURCE_FILES1
  readme.md
)

set_target_properties(tvm_model PROPERTIES
  FRAMEWORK TRUE
  FRAMEWORK_VERSION A
  MACOSX_FRAMEWORK_IDENTIFIER cn.tvm.tvm_model
  # MACOSX_FRAMEWORK_INFO_PLIST Info.plist
  # "current version" in semantic format in Mach-O binary file
  VERSION 1.0.0
  # "compatibility version" in semantic format in Mach-O binary file
  SOVERSION 1.0.0
  PUBLIC_HEADER "${HEADER_FILES1}"
  RESOURCE      "${RESOURCE_FILES1}"
  # XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY "iPhone Developer"
)
   ```
   然后，做一个build.sh,以便能反复编译(只编译模拟器版本)：
   ```
#!/bin/bash

[ -d "build" ] && rm -R build/
mkdir build
cd build

cmake .. -G Xcode -DCMAKE_TOOLCHAIN_FILE=../ios.toolchain.cmake -DPLATFORM=SIMULATOR64

cmake --build . --config Debug
cmake --build . --config Release

cd ..
   ```
   
   最后，把参考3内的example app复制一份，修改一下，引用我们的tvm_model.framework，就能在模拟器上跑起来了。
  
## 版本3: opencv2.framework
   要在tvm_model.framework内使用 opencv,就需要连接opencv库.
   
   ```
   # include 路径
   include_directories(SYSTEM ../OpenCV/3.4.3/opencv2.framework)
   # link opencv2 库
   target_link_libraries(tvm_model "-framework opencv2")
   # link 的搜索目录
   set_target_properties(tvm_model PROPERTIES
      LINK_FLAGS "-Wl,-F../OpenCV/3.4.3"
   )

   ```
   
   在 `tvm_model_c.cc` 内调用一下 opencv 函数，做个简单测试:
   
   ```
   
#include <opencv2/core/core.hpp>
#include <opencv2/opencv.hpp>

#include <vector>

    static int Mat_to_HWC(const cv::Mat &frame, int w, float *ret){
        assert(ret && !frame.empty());
        if(! ret || frame.empty())
            return -1;

        unsigned int volChl = w * w;

        for (unsigned j = 0; j < volChl; ++j) {
            for (int c = 0; c < 3; ++c) {
                ret[j * 3 + c ] = static_cast<float>(float(frame.data[j * 3 + c]) / 255.0);
            }
        }
        return 0;
    }


int TVMModelC::run(int x){
	cv::Mat img;
	std::vector<float> ret(128);
	Mat_to_HWC(img, 64, &ret[0]);
	return x*x;
}

   ```
   
   编译一下:
   ```
    sh build.ios-xcodeproj.sh  # 创建xcode工程
    sh build.ios.sh #编译
   ```
   编译结果文件是：
   ` build.ios/Debug-iphonesimulator/tvm_model.framework/tvm_model`
   大小是1.4M.

## 用cmake编译 ios app
   ### 参照参考10，能生成 ios app.
   ```
	cmake_minimum_required(VERSION 3.14)

	Project (tvm_demo_ios C CXX)

	set(SRC_FILES tvm_demo_ios/main.m)

	add_executable(tvm_demo_ios MACOSX_BUNDLE ${SRC_FILES})

	include_directories(SYSTEM ../tvm_model/build.ios/Debug-iphonesimulator/tvm_model.framework)

	target_link_libraries(tvm_demo_ios "-framework Foundation  -framework UIKit -framework tvm_model")

	set_target_properties(tvm_demo_ios PROPERTIES
	  LINK_FLAGS "-Wl,-F../tvm_model/build.ios/Debug-iphonesimulator"
	  MACOSX_BUNDLE_GUI_IDENTIFIER "com.example.\${PRODUCT_NAME:identifier}"
	)
   
	install(TARGETS tvm_demo_ios DESTINATION tvm_demo_ios.app)
   ```
   为了编译 ios app,需要连接到 UIKit.framework:
   `   target_link_libraries(tvm_demo_ios "-framework UIKit")`

   ### 添加 MACOSX_BUNDLE_GUI_IDENTIFIER 属性
      上面的cmake文件，能编译通过，但不能被启动运行: Bundle id can't be determined。
      参照参考11，添加 MACOSX_BUNDLE_GUI_IDENTIFIER  属性，才能在ios模拟器内，运行起来。
      `  MACOSX_BUNDLE_GUI_IDENTIFIER "com.example.\${PRODUCT_NAME:identifier}")`

   ### 复制 resource
   把 data 目录，作为 resource, 打包进 app:
   ```
	# copy resource phase

	set(APP_NAME \${TARGET_BUILD_DIR}/\${FULL_PRODUCT_NAME})
	set(RES_DIR  ./data)
	add_custom_command(
	    TARGET tvm_demo_ios
	    POST_BUILD
	    COMMAND mkdir ${APP_NAME}/Resources/ && echo "." || echo "."
	    COMMAND cp -Rf ../resources/*  ${APP_NAME}/Resources/
	)   
   ```
   cmake把自定义脚本，生成在一个文件内，在xcode编译的时候，会调用这个文件：   
   `tvm_demo_ios/build.ios/CMakeScripts/tvm_demo_ios_postBuildPhase.makeDebug`.
   脚本运行的当前目录是:
   `tvm_demo_ios/build.ios`.

## 参考
  - 参考1: [cmake 打包 ios sdk](https://github.com/Tencent/ncnn/wiki/cmake-%E6%89%93%E5%8C%85-ios-sdk?from=singlemessage)
  - 参考2: [ios使用cmake编译framework](https://blog.csdn.net/zhuyunier/article/details/83025615)
  - 参考3: [ios-cmake](https://github.com/leetal/ios-cmake) 
  - 参考4: [cmake-3.14](https://cmake.org/download/) 
  - 参考5: [What is the difference between Embedded Binaries and Linked Frameworks](https://stackoverflow.com/questions/32375687/what-is-the-difference-between-embedded-binaries-and-linked-frameworks)
  - 参考6： [Technical Note TN2435: Embedding Frameworks In An App](https://developer.apple.com/library/archive/technotes/tn2435/_index.html)
  - 参考7: [Why I cannot link the Mac framework file with CMake?](https://stackoverflow.com/questions/17070101/why-i-cannot-link-the-mac-framework-file-with-cmake)
  - 参考8: [CMake Build Configuration for iOS](https://github.com/sheldonth/ios-cmake)
  - 参考9： [CMake include_directories skips OS X frameworks](https://stackoverflow.com/questions/33332021/cmake-include-directories-skips-os-x-frameworks)
  - 参考10: [cmake ios app 跨平台编译的经验之谈－－Xcode之旅](https://www.jianshu.com/p/e4ece730de8e)
  - 参考11: [How to set up CMake to build an app for the iPhone](https://stackoverflow.com/questions/822404/how-to-set-up-cmake-to-build-an-app-for-the-iphone)