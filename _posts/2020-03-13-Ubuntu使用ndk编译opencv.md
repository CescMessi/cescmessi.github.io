---
layout:     post
title:      Ubuntu使用ndk编译opencv
subtitle:   解决官方sdk不能使用最新版ndk的问题
date:       2020-03-13
author:     CescMessi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Android
    - Ubuntu
---

# 背景说明
ndk从16开始默认不使用gnustl_static模板库，从18开始直接取消了，而opencv的Android sdk还没有跟上这个更新，因此要在Android上用ndk使用opencv，需要用旧版ndk，比如ndk14。但是如果想使用最新的ndk，也不是没有办法，那就是自己编译。

# 准备工具
- cmake. 这里我使用的是gui版本，个人感觉比较方便，可以直接使用apt安装`sudo apt install cmake-qt-gui`
- Android Studio. 其实主要是需要Android SDK和NDK，如果你用单独的SDK也没有问题。这里有一个坑**不要使用21.0.6113669版本的ndk**，使用这个版本的ndk编译会出现问题，具体看[Github Issues](https://github.com/android/ndk/issues/1179)，使用前一个版本20是正常的，或者等bug修复。
- opencv源码。直接上[Github](https://github.com/opencv/opencv)上下载一份即可。我使用的是4.2.0.

其他的ant，Python，clang不确定是否必须，NDK本身已经内置了一个clang，个人认为是不需要的。

# 开始编译
首先打开cmake的gui，直接在启动器能找到，或者命令行输入`cmake-gui`，在前两行分别选择源码的文件夹和输出的文件夹。
![cmake设置1](https://i.loli.net/2020/03/13/vFRGHtbnIlVL8sM.png)

之后点击add entry，需要添加的有以下几项：
- ANDROID_STL，用来设置模板库，STRING类型，这里填`c++_shared`
- ANDROID_ABI，STRING类型，添加自己需要的abi即可，abi具体列表可见[Android ABI](https://developer.android.com/ndk/guides/abis)
- ANDROID_SDK，PATH类型，填写你的Android SDK位置


![深度截图_选择区域_20200313225547.png](https://i.loli.net/2020/03/13/XxEoRvmQSh8FNlz.png)

随后点击Configure，如果输出文件夹不存在会弹窗让你创建，yes即可。之后的弹窗需要选择生成工具，选择Unix Makefiles和Specific toolchain file for cross-compiling。

![cmake设置3](https://i.loli.net/2020/03/13/ueNsrLyXVD3Aivd.png)

点击next，需要设置toolchain file的位置，这个文件位于ndk目录下，`ndk/20.1.5948944/build/cmake/android.toolchain.cmake`，其中20.1.5948944是ndk版本号。

![cmake设置3](https://i.loli.net/2020/03/13/FtxI6gTqGCzXPJ9.png)

点击finish如无意外会看到以下界面，最后一行会显示Configuring done。

![cmake设置4](https://i.loli.net/2020/03/13/xcy3d81Y9irlBOQ.png)

接下来可以根据需要自行对entry进行修改，我取消了生成Android example和Android Project，因为需要Android SDK生成项目比较耗时，而且有可能会出错。选择完成后点击Generate，没有意外的话会很快显示Generating done。

之后命令行进入到输出目录，输入命令`make`，或`make -j8`使用8线程，坐和放宽，稍等片刻，这取决于你机器的性能。完成后输入`make install`

![深度截图_选择区域_20200313234701.png](https://i.loli.net/2020/03/13/HmNaC7i1qzKDcEJ.png)
![深度截图_选择区域_20200313234750.png](https://i.loli.net/2020/03/13/SThfKkb5VBCUuMg.png)

编译好的文件在install文件夹中，jni目录的路径为`install/sdk/native/jni`，使用可以参考[OpenCV On Android最佳环境配置指南(Android Studio篇)](https://www.jianshu.com/p/6e16c0429044)。

# 参考资料
[Windows环境下编译适用于NDK18及以上版本的opencv库](https://www.qingtingip.com/h_235673.html)