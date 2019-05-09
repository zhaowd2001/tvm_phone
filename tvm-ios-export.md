tvm: from_tensorflow_for_ios

[export lib for ios](https://github.com/zhaowd2001/tvm_phone/blob/master/tvm-ios-export.md)

2019/5/9
  目录
  
[TOC]

## 代码下载 

   [ios deploy](https://zwd.3wfocus.com/svn/files/trunk/tp/tvm/apps/ios_deploy)

   ![ios deploy Demo](https://github.com/zhaowd2001/tvm_phone/blob/master/tvm-ios-export.png?raw=true)
   
   
   
## 目的

   本文描述如何在 mac 下，从tensorflow pb模型文件，生成 iphone 手机上的 dylib文件。
   这样，就能在iphone手机上，调用 模型，进行推导。

## 从tensorflow生成 dylib

   参见参考3，能从pb模型文件，生成dylib文件。
   把python文件的前半部分复制出来，改一下，就能生成 dylib文件.
   做成文件 `from_tensorflow_ios.py`.

### 导入 tensorflow pb 
   
   ```
   shape_dict = {'DecodeJpeg/contents': x.shape}
   dtype_dict = {'DecodeJpeg/contents': 'uint8'}
   sym, params1 = relay.frontend.from_tensorflow(graph_def, layout=layout, shape=shape_dict)
   ```

### 设置 target
   参照参考2，改动一下 target 的设置:

   - gpu

   ```
   target = "metal"
   target_host = "llvm -target=arm64-apple-darwin"
   ```

   - cpu
   
   ```
   target_host = "llvm -target=arm64-apple-darwin"
   target = target_host
   ```

### 设置 layout
   `layout = 'NCHW'`

### 编译模型

   ```
   print(" Relay build ...")
   with relay.build_config(opt_level=3):
     graph, lib, params = relay.build(sym, target=target, target_host=target_host, params=params1)
   ```

### codesign
   tvm编译xcode dylib的时候，会要求你设置 TVM_IOS_CODESIGN.
   我只是在iphone模拟器内运行app，还没有签名的信息。
   可以修改一下tvm py代码，跳过codesign:
   
   ```
   ./python/tvm/contrib/xcode.py:
   def codesign(lib):
    if "TVM_IOS_CODESIGN" not in os.environ:
        print("TVM_IOS_CODESIGN not set, skip codesign")
        return

   ```
   
### 生成 dylib
   生成ios能调用的dylib库：
   
   ```
   arch = "arm64"
   sdk = "iphoneos"
   from tvm.contrib import util, xcode
   path_dso1='./pb.dylib'
   lib.export_library(path_dso1, xcode.create_dylib, arch=arch, sdk=sdk)
   xcode.codesign(path_dso1)
   ```
 
## 参考
   - 参考1: [ios rpc](https://github.com/dmlc/tvm/tree/master/apps/ios_rpc)
   - 参考2: [ios_rpc_test.py](https://github.com/dmlc/tvm/blob/master/apps/ios_rpc/tests/ios_rpc_test.py)
   - 参考3：[Compile Tensorflow Models](https://docs.tvm.ai/tutorials/frontend/from_tensorflow.html#sphx-glr-tutorials-frontend-from-tensorflow-py)
