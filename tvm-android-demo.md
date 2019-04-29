# tvm-android-demo
tvm android arm gpu opencl

2019.4.29

## 目的

   TVM 自带的 android demo, 编译时有以下问题:

   问题1. 需要用到 android ndk, opencl 头文件和opencl库文件。
   问题2. 因为用build.sh调用ndk-build,编译必须在linux下编译
   问题3. 编译还需要 tvm jar:[tvm4j-core-0.0.1-SNAPSHOT.jar]()
   
   这导致编译过程比较痛快，不花费一些时间，很难编译出app.

   所以，我把自己编译的apk放出来，如果你只想在android上体验上一下 tvm 的速度，可以下载本app。

   我把依赖的opencl/tvm jar,都放到了源代码内，这样你只要用android studio就可以编译出本app.
 
 
## 下载
   下载网站用户名口令都是pub
   [tvm android demo apk](https://zwd.3wfocus.com/!/#files/view/head/trunk/tp/tvm/apps/android_deploy/app/build/outputs/apk/debug/app-debug.apk)
   android demo 源代码 [tvm android demo](https://zwd.3wfocus.com/!/#files/view/head/trunk/tp/tvm/apps/android_deploy)
   [tvm4j-core-0.0.1-SNAPSHOT.jar](https://zwd.3wfocus.com/!/#files/view/head/trunk/tp/tvm/apps/android_deploy/app/src/main/libs)
   [libOpenCL.so](https://zwd.3wfocus.com/!/#files/view/head/trunk/tp/tvm/apps/android_deploy/app/src/main/jni/opencl/jnilibs)
   [libtvm4j_runtime_packed.so](https://zwd.3wfocus.com/!/#files/view/head/trunk/tp/tvm/apps/android_deploy/app/src/main/jnilibs)
   


## 代码功能
1. 能选择CPU还是GPU，然后再加载 tvm 运行库
2. 选择图片后，用tvm进行推理，然后显示推理花费的时间

## 改动点
1. 自带了90M模型文件，编译的时候不用下载了
2. 命令行 build.sh 编译 libtvm4j_runtime_packed.so 变成了 CMakeLists.txt
3. 加入了 opencl 头文件和 so. libOpenCL.so 是从华为P10手机内复制出来的
4. 自带了 tvm4j-core-0.0.1-SNAPSHOT.jar, 不用再下载了
5. 自带了 libtvm4j_runtime_packed.so, 也不用编译so了
6. 这tvm模型的两个 so 是 arm64, 所以我只编译了 arm64.
   `build.gradle: abiFilters 'arm64-v8a'`
   tvm模型的两个so: 
   `apps\android_deploy\app\src\main\assets\deploy_lib_cpu.so/deploy_lib_opencl.so`
