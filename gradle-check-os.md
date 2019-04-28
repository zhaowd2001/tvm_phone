#gradle内检测os类型
##目的
AndroidStudio 用 gradle 来编译android 程序, 有时候调用批处理来执行一些命令。
为了能在windows和linux两个环境下，能执行不同的批处理，就需要能判断当前操作系统的类型。
##TVM的android demo
TVM自带的[android demo](#ref1)内面，为了编译出 tvm so，就调用了android ndk的ndk-build命令。

```javascript
task buildJni(type: Exec, description: 'Build JNI libs') {
    commandLine 'sh', 'src/main/jni/build.sh'
}
```
问题是，上面的命令是linux下的sh批处理.
为了在windows上运行，需要改成下面的样子:
```javascript
task buildJni(type: Exec, description: 'Build JNI libs') {
    commandLine 'cmd.exe', '/c src\\main\\jni\\build.bat'
}
```
##合二为一
为了能同时在windows和linux编译 tvm android demo,可以把上面的代码，加上操作系统判断，合并在一起，最后变成下面这样:
```javascript
task buildJni(type: Exec, description: 'Build JNI libs') {
    if (System.properties['os.name'].toLowerCase().contains('windows')) {
        commandLine 'cmd.exe', '/c src\\main\\jni\\build.bat'
    } else {
        commandLine 'sh', 'src/main/jni/build.sh'
    }
}
```




##参考
###参考1 TVM自带的[android demo](https://github.com/zhaowd2001/tvm/tree/master/apps/android_deploy)
###参考2 [How to detect the current OS from gradle ](https://stackoverflow.com/questions/11235614/how-to-detect-the-current-os-from-gradle)
