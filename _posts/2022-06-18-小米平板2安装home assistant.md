---
layout:     post
title:      小米平板2安装home assistant
subtitle:   使用Lineage OS + Termux + Ubuntu
date:       2022-06-18
author:     CescMessi
header-img: https://c1.mifile.cn/f/i/15/mipad2/specs-screen-bg.jpg
catalog: true
tags:
    - Android
    - Ubuntu
    - Home Assistant
---

# 背景
20年出于兴趣购买了一部二手的小米平板2，装上了Windows 10和Lineage OS双系统，由于性能比较差，平时也就看看视频和电子书用一下。最近突然觉得，家里的一些米家智能设备确实还不够智能，比如我想回家之后摄像头自动关闭，热水器停止加热之后直接断开电源……这些操作在米家APP里都无法实现，而在Home Assistant里，这些都可以通过自动化脚本轻松实现，于是萌生了安装Home Assistant的念头。但笔记本要带去公司用，路由器性能弱磁盘小，于是我把目光看向了经常吃灰的小米平板2。

# 准备工作
## 操作系统
操作系统我使用的是[Lineage OS](https://forum.xda-developers.com/t/unofficial-lineageos-13-0-for-xiaomi-mi-pad-2.3565760/)，这是XDA大神编译的x86架构Android 6.0的系统，个人感觉比官方的MIUI要更加流畅，使用起来也更加自由。当然理论上MIUI也是不影响后续的操作的。

## Termux
由于Android自带的Linux环境是不完整的，而Home Assistant是基于Python开发的，考虑到可能调用底层的C库，因此还是使用完整的Linux环境比较保险。比较合适的路线就是Termux内安装Linux。

我使用的Termux是Google Play上直接下的0.83版，这个版本支持的操作系统为Android 5.0以上，所以理论上MIUI和Lineage OS都是可以安装的。

由于Termux官方已经不再维护Termux老版本的源，因此安装完Termux后是无法直接使用pkg安装软件的。根据[Github](https://github.com/termux/termux-packages/issues/4467#issuecomment-572008515)上的提示，我们可以下载仓库的存档包，然后在自己的电脑上搭建仓库，修改源即可继续使用pkg。当然你也可以直接在里面搜索想要的软件，然后手动安装，但需要自己解决依赖问题。为了方便，我将`termux-repositories-legacy-24.12.2019.tar`解压后利用Everything搭建了HTTP服务器，之后再在Termux上修改源的地址指向我的电脑，就可以正常使用了。例如我解压后在`F:\repo`目录下有个`termux-repositories-legacy`目录，我电脑的ip是192.168.31.253，我的Everything HTTP服务端口为2334，那么就可以将`/data/data/com.termux/files/usr/etc/apt/sources.list`改为以下内容：
```
# The main termux repository:
deb http://192.168.31.253:2334/F%3A/repo/termux-repositories-legacy/webroot/termux-packages stable main
deb http://192.168.31.253:2334/F%3A/repo/termux-repositories-legacy/webroot/science-packages-21 science stable
deb http://192.168.31.253:2334/F%3A/repo/termux-repositories-legacy/webroot/game-packages-21 games stable
deb http://192.168.31.253:2334/F%3A/repo/termux-repositories-legacy/webroot/termux-root-packages-21 root stable
deb http://192.168.31.253:2334/F%3A/repo/termux-repositories-legacy/webroot/unstable-packages-21 unstable main
deb http://192.168.31.253:2334/F%3A/repo/termux-repositories-legacy/webroot/x11-packages-21 x11 main
```
随后使用`pkg update`命令，即可在Termux内安装想要安装的软件了。推荐安装`openssh`和`tsu`，前者可以用`sshd`命令启用一个8022端口的ssh服务，方便在电脑上操作；后者可以使用root权限，用起来更自由一些。

## Ubuntu
安装完Termux后就可以安装Ubuntu了，这里我使用的是AnLinux提供的脚本：
```bash
$ pkg install wget openssl-tool proot -y && hash -r && wget https://raw.githubusercontent.com/EXALAB/AnLinux-Resources/master/Scripts/Installer/Ubuntu/ubuntu.sh && bash ubuntu.sh
```
运行完成后运行start-ubuntu.sh即可进入Ubuntu系统。但是这个系统是没有整机的root权限的，一些端口比如22无法打开，因此我一般先用tsu获取Android系统的root权限后再执行启动脚本。
```bash
tsu
./start-ubuntu.sh
```

进入Ubuntu系统后就是常规的Linux操作了，如果网络不好可以先把源换成清华的源。同样在Ubuntu系统里面我也推荐安装ssh服务端，不然在平板上点来点去太费劲了，这里就不赘述怎么安装了，注意下密钥登录要把严格模式关了就好。总之，把Ubuntu系统环境简单初始化成你喜欢的样子就可以下一步操作了。

# Home Assistant安装
Home Assistant安装使用的是官网的[Home Assistant Core 安装教程](https://www.home-assistant.io/installation/linux#install-home-assistant-core)。

首先根据教程安装依赖项：
```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y python3 python3-dev python3-venv python3-pip libffi-dev libssl-dev libjpeg-dev zlib1g-dev autoconf build-essential libopenjp2-7 libtiff5 libturbojpeg0-dev tzdata
```

然后创建新用户和Python虚拟环境，由于我们的Ubuntu与实际还是有所区别的，所以要略微修改一下：
```bash
useradd -rm homeassistant
su - homeassistant
bash
mkdir /srv/homeassistant
cd /srv/homeassistant
python3 -m venv .
source bin/activate
```

此时已经激活了Python虚拟环境，应该可以看到命令行的样式为`(homeassistant) homeassistant@localhost:~$`，此时安装需要的包：
```
python3 -m pip install wheel
```

教程里这一步后是直接安装了，但由于Lineage OS是32位的操作系统，安装的Python版本也是Python 3.8 x86，一些Python包并没有提供编译好的二进制文件，而是需要根据源码编译。此时就涉及到了编译器的问题。gcc和g++应该是Ubuntu里面已经有了，如果没有就apt先装一下。另外还需要安装rust编译器，这个运行`curl https://sh.rustup.rs -sSf | sh`就可以了。

随后就可以安装了：
```
pip3 install homeassistant
```

这一步可能会耗费比较久的时间，因为大部分包都需要编译，而小米平板2的CPU不能算强，建议没事把屏幕点亮看看，防止睡眠睡死了。

完成之后，就可以运行命令启动Home Assistant了：
```bash
hass
```

访问`http://X.X.X.X:8123`(X.X.X.X为你的小米平板2的局域网ip地址)，应该可以看到引导界面。注意看看命令行的日志，可能会初始化比较久，主要与网络有关，有的插件太久装不上直接ctrl+c中断重来其实也不影响后面使用。

在确认Home Assistant安装成功后，以后运行就可以直接放后台了，我一般这样启动：
```bash
# ssh连入Ubuntu的root用户
# 切换到homeassistant用户
su - homeassistant
bash
# 激活Python虚拟环境
source /srv/homeassistant/bin/activate
# 运行hass
nohup hass > log &
# 此时可以直接关掉ssh窗口，hass继续运行
```

# HACS安装
Home Assistant一个重要的插件就是[HACS](https://hacs.xyz/)，通过HACS可以安装github上各种有用的插件。但HACS官方提供的命令并不能直接安装，主要是因为Lineage OS是32位系统，Python版本最高为3.8，而Home Assistant最新版本已经不支持Python 3.8了，最新版的HACS又不支持我们使用的Home Assistant版本，所以需要修改一下HACS安装脚本，安装旧版的HACS。

首先确保Ubuntu系统安装了`wget`和`unzip`，没有的话用apt装一下。

然后下载安装脚本
```bash
wget -O - https://get.hacs.xyz
```

此时能看到当前目录下会有一个get的文件，利用vim等文本编辑器打开文件，找到59行的`wget "https://github.com/hacs/integration/releases/latest/download/hacs.zip"`,将其改为`wget "https://github.com/hacs/integration/releases/download/1.24.5/hacs.zip"`，保存退出后，运行以下命令开始安装：
```bash
chmod +x get
bash get
```

安装时间与网络状况有关，安装完成后，就可以根据HACS官网的介绍进行进一步的配置和操作了。

# 最终成果
![Img](https://raw.githubusercontent.com/CescMessi/CescMessi.github.io/master/img/Snipaste_2022-06-18_12-54-19.png)
