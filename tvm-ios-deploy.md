tvm: ios_deploy

[ios deploy](https://github.com/zhaowd2001/tvm_phone/blob/master/tvm-ios-deploy.md)

2019/5/9
  目录
  
[TOC]

## 代码下载 

   [ios deploy2 ](https://zwd.3wfocus.com/svn/files/trunk/tp/tvm/apps/ios_deploy2)

   ![ios deploy2 Demo](https://github.com/zhaowd2001/tvm_phone/blob/master/tvm-ios-deploy2.png?raw=true)
   
   
   
## 目的

   本文描述如何制作 ios app, 调用tvm，加载模型文件，进行推导.

## 代码准备
   把参考1内的代码，复制一份，改名成 ios_deploy2.

   把rpc相关代码删除，只保留app框架。

   
## 参考
   - 参考1: [ios rpc](https://github.com/dmlc/tvm/tree/master/apps/ios_rpc)
   - 参考2：[Compile Tensorflow Models](https://docs.tvm.ai/tutorials/frontend/from_tensorflow.html#sphx-glr-tutorials-frontend-from-tensorflow-py)
