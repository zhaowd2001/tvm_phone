tvm: from_tensorflow_ios

[export lib for ios](https://github.com/zhaowd2001/tvm_phone/blob/master/tvm-ios-export.md)

2019/5/9
  目录
  
[TOC]

## 代码下载 

   [from_tensorflow_ios.py](https://zwd.3wfocus.com/svn/files/trunk/tp/tvm/apps/ios_deploy)

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
   
   要使用ios metal，需要在tvm内启用metal，方法如下：
   - 修改 build/config.cmake, 把 `set(USE_METAL OFF)`, 改成 `set(USE_METAL ON)`.
   - 重新编译 tvm: 
   ```
   cd build
   cmake ..
   make -j4
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

### 保存模型数据
   只有 dylib ，tvm还需要模型文件才能运行起来。
   需要把graph 和 params保存下来，代码如下:
   
   ```
        so_name = "cpu_lib"

        lib.save(os.path.join(build_dir, so_name + ".o"))

        with open(os.path.join(build_dir, 'cpu_deploy_graph.json'), 'w') as f_graph_json:
            f_graph_json.write(graph)

        with open(os.path.join(build_dir, 'cpu_deploy_param.params'), 'wb') as f_params:
            # save the parameters as byte array
            param_bytes = tvm.relay.save_param_dict(params)
            f_params.write(param_bytes)
   ```
   
   
## 文件信息
   下面总结一下，模型文件信息，和为ios生成的 dylib信息。
   
   |序号|文件名|大小|用途|
   |--|--|--|--|
   |1|~/.tvm_test_data/tf/InceptionV1/classify_image_graph_def-with_shapes.pb| 91M |tensorflow模型|
   |2|~/.tvm_test_data/data/imagenet_2012_challenge_label_map_proto.pbtxt| 63K |tensorflow模型描述|
   |3|~/.tvm_test_data/data/elephant-299.jpg| 53K |测试图片
   |4|~/.tvm_test_data/data/imagenet_synset_to_human_label_map.txt| 724K |分类文件
   |5|<tvm_home>/apps/ios_deploy/from_tensorflow.py||mac上,从模型生成 dylib|
   |6|<tvm_home>/apps/ios_deploy/from_tensorflow_ios.py||mac上，从模型生成 dylib,能在ios上运行|
   |7|<tvm_home>/apps/ios_deploy/build/metal_lib.dylib | 1M |ios上运行的 dylib, 能调用模型, gpu |
   |8|<tvm_home>/apps/ios_deploy/build/metal_deploy_graph.json | 170K |模型结构 |
   |9|<tvm_home>/apps/ios_deploy/build/metal_deploy_param.params | 91M |模型参数 |
   |10|<tvm_home>/apps/ios_deploy/build/cpu_lib.dylib | 490K |ios上运行的 dylib, 能调用模型, cpu |
   |11|<tvm_home>/apps/ios_deploy/build/cpu_deploy_graph.json | 179K |模型结构 |
   |12|<tvm_home>/apps/ios_deploy/build/cpu_deploy_param.params | 91M |模型参数 |
   
   
## 参考
   - 参考1: [ios rpc](https://github.com/dmlc/tvm/tree/master/apps/ios_rpc)
   - 参考2: [ios_rpc_test.py](https://github.com/dmlc/tvm/blob/master/apps/ios_rpc/tests/ios_rpc_test.py)
   - 参考3：[Compile Tensorflow Models](https://docs.tvm.ai/tutorials/frontend/from_tensorflow.html#sphx-glr-tutorials-frontend-from-tensorflow-py)
