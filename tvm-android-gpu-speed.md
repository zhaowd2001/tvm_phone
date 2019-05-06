# [tvm 在 android gpu 上的速度问题](https://github.com/zhaowd2001/tvm_phone/blob/master/tvm-android-gpu-speed.md)
tvm android gpu opencl
2019/5/6

@[toc]
## 背景
   tvm 当前版本是 0.6.dev.
   
   android手机上，gpu是由 [opencl](https://baike.baidu.com/item/OpenCL/8477301?fr=aladdin) 来驱动的。

   在带有gpu的android手机上，可以用tvm来调用 opencl 做模型推导([例子](https://github.com/zhaowd2001/tvm_phone/blob/master/tvm-android-demo.md))。


## gpu的问题1：第一次推理慢
   tvm加载模型后，用opencl进行第一次推理的时候，会非常慢. 5M的模型，需要40秒.
   
   用cpu的话，就没这个问题。
   
   网上一种说法是，opencl在第一次运行的时候，需要动态编译指令。
   
   
## gpu的问题2: get_output 慢
   tvm调用opencl api来进行模型推理运算，确实很快(几个毫秒)。
   
   但要从gpu内取出推理结果，却很慢很慢(128个数据，需要200毫秒).
   
   gpu运算的结果，是存在gpu 内存内的.
   
   要读取gpu内存，需要调用 opencl api，把数据复制到cpu内存才行.
   
   这个时候，opencl api会很慢很慢.
   
   网上一种说法是，此时opencl在等待gpu运算完毕，才能复制出gpu 内存。
   
   无论是什么原因，反正很慢.
   
   并且由于opencl是各手机厂商提供的库，其他人也没法修正.
   参见末尾的参考1和参考2.
   
## gpu的问题3: tvm崩溃
   根据我的测试，要运行tvm cpu/gpu，android版本要高(8.0).
   低版本的android，tvm直接崩溃，而不是返回错误码。
   我没法提前知道那些android系统能运行tvm.
   
   这导致一个很讨厌的问题：我不能在主进程内直接调用tvm，万一tvm崩溃了，整个app就异常退出了.
   
   回避办法：我需要做一个android service，放到另一个进程运行tvm,测试一下tvm能否正常工作。
   
## 结论
   tvm 目前是0.6.dev，还在开发阶段。
   
   对老手机(android 5/6, armv7)，运行会崩溃. 
   
   android 手机上，即使不使用 tvm gpu，还是可以使用 tvm cpu的。
   
## 参考
   参考1: [Any tips on copying data from cpu to gpu, bottleneck?](https://discuss.tvm.ai/t/any-tips-on-copying-data-from-cpu-to-gpu-bottleneck/780)
   
   参考2: [How to make the GPU to CPU memory copy faster? #979](https://github.com/dmlc/tvm/issues/979)
