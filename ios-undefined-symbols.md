# [ios： Undefined symbols](https://github.com/zhaowd2001/tvm_phone/blob/master/ios-undefined-symbols.md)
ios universal build

2019.4.30
## 背景
   用xcode 做了一个iphone ui demo工程，它依赖另一个 tvm_model.framework 工程.

   demo内使用了 tvm_model.framework内的c++类 TVMModel.
   
## 问题
   编译demo工程，Debug能编译通过，Release总是报告错误:
   Undefined symbols for architecture arm64/armv7:
  "_OBJC_CLASS_$_TVMModel", referenced from: objc-class-ref in AppDelegate.o

## 调查过程
   在xcode tvm_model工程设置内，对比了Debug和Release区别，感觉有3个选项和编译有关:

|Options                       |Debug|Release|
|--|--|--|
|GCC_GENERATE_DEBUGGING_SYMBOLS|YES  |NO     |
|GCC_OPTIMIZATION_LEVEL        |0    |s      |
|GCC_SYMBOLS_PRIVATE_EXTERN    |NO   |YES    |
|Build Active Architecture Only|YES  |NO     |
   把Release的4个配置项，挨个修改尝试，发现GCC_SYMBOLS_PRIVATE_EXTERN从YES改成NO，demo就能编译成功.
   把Build Active Architecture Only从NO改成YES也能成功。

## 分析
   定位到原因，上网搜索了一下，GCC_SYMBOLS_PRIVATE_EXTERN对应的是:
   `Build settings` 内的 `Symbols Hidden By Default`.

   从字面意思看，是把符号隐藏起来(能让编译结果缩小很多).

   Release模式，所有符号被隐藏起来了，那么demo就连接不到tvm_model内的类了.

## 解决办法
   只要把需要导出的类，明确的告诉编译器就行了。
   
   具体参见这个链接[制作framework](https://blog.csdn.net/u012234115/article/details/51849726)
   
   还有一个问题，就是demo连接的device是i386的模拟器，而tvm_model.framework只编译了armv7/armv8平台的代码。

   必须把tvm_model工程的Build Active Architecture Only，设置成YES，这样才能和demo一样，编译出模拟器需要的i386代码。
   
