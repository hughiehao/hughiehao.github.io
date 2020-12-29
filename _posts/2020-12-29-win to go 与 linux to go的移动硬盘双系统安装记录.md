---
layout: post
title: 'win to go 与 linux to go的移动硬盘双系统安装记录'
date: 2020-12-29
author: hughie
cover: '/assets/img/Linux-and-windows.png'
tags: wtg ltg 系统安装 开机引导
---

# 前言

WTG(Win To Go)项目是微软官方的，可以将系统装进u盘的便携系统，这极大方便了需要临时使用自己系统的人员，可以在其他电脑上插上u盘使用u盘内的系统，不同于普通系统安装的方法，安装进移动介质就能直接在电脑上启动使用。

LTG顾名思义，是Linux的移动版本，但是安装时按普通安装方式安装的，跟WTG有区别；Linux本身占用内存就小，非常适合移动介质使用。

安装双系统后使用rEFInd引导工具美化引导界面。

**环境：**

Windows10企业版

  

<br />

#### 一、准备工作：

**1、WTG**

根据微软官方的意见，要在U盘上安装WTG，至少需要满足以下几点：

-   **高速读写，接口为USB 3.0以上**。一般3.0接口的普通u盘就可以正常使用安装使用了；
-   **32GB及以上的空间**；容量大点更好，并且不局限于u盘，移动硬盘更好；
-   **Windows 10 Enterprise（企业版）镜像**。首先WTG本身是个企业版功能，并可通过企业版、专业版和教育版进行部署。

**2、LTG**

Linux什么系统版本都可，我安装的是ubuntu18.04。

**3、相关工具**

[WTG辅助工具](https://bbs.luobotou.org/thread-761-1-1.html)

[Rufus烧录软件](https://github.com/pbatard/rufus/releases/download/v3.13/rufus-3.13.exe)

[rEFInd引导工具](https://newcontinuum.dl.sourceforge.net/project/refind/0.11.4/refind_0.11.4-1_amd64.deb)

[win10系统iso（支持WTG的版本）](https://msdn.itellyou.cn/)

[Linux系统iso](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/18.04/ubuntu-18.04.5-desktop-amd64.iso)

8G以上u盘（烧录linux启动盘）

128G以上u盘（或其他安装双系统的移动设备）

<br />

#### 二、安装步骤

**1、分区**

由于我装的是双系统，所以在移动硬盘是先分了两个区，分别用于安装两个系统。

**2、WTG安装**

安装时有三种方式：win10自带的WTG创建工具、Rufus、WTG辅助工具。

1.  自带工具安装

    win10系统是专业版（Pro)、企业版（Enterprise）或教育版（Education），可以直接在cortana搜索框中搜索“windows to go”，会出现一个创建工具；

2.  Rufus烧录

    软件打开界面选择

    -   安装时可选是windows还是windows to go；
    -   然后扇区(Partition scheme）建议选GPT，老一点的电脑可以选MBR（不然没法引导启动）；
    -   高级选项（Advanced drive properties）里建议把”显示USB外置硬盘“勾上，方便作普通存储介质；
    -   文件系统（File System）选NTFS，不选FAT32或exFAT，那是给U盘用的。

3.  辅助工具安装

    辅助工具安装非常方便，同时能自动解决win10 1809版本缺失文件wpprecorder.sys，导致蓝屏的问题，推荐使用此方法，软件使用界面选择：

    -   选择iso文件；
    -   选择待安装系统的硬盘设备；
    -   选择安装版本（一般企业版好）；
    -   高级选项-常用，选择UEFI+GPT；
    -   高级选项-分区，调整EFI分区大小（建议为350~500），调整硬盘分区大小。

**3、LTG安装**

首先设置完WTG后重启机器，再安装LTG。

使用Rufus烧录Linux iso文件到u盘中，软件选择项与WTG中Rufus方式安装相同。

Bios中设置u盘启动，进入Linux安装界面：

-   安装类型，选择其他选项；
-   找到分配给Ubantu的空间，选中，点击下面的“ **-** ” 号，等待系统处理，前缀变成空闲；
-   选中该空闲区域，点击下面的“ **+** ”号；创建主分区，可以将大部分空间分配到主分区，只留下一定大小的空间余量用于交换空间即可（交换空间设置跟内存大小相同的空间），如共64G空间，内存是8G，这里就分配56G给主分区，选择挂载点" / ”。点击OK；
-   选中剩下的空闲空间，点击加号，选择主分区，交换空间。点击OK；
-   **（重要）**在下方安装启动引导器的设备选项中，选择整个移动硬盘（没有单独划分efi引导分区）。点击现在安装。

<br />

#### 三、启动项配置

rEFInd一般在UEFI启动环境下使用，他可以用来引导各类操作系统的启动，所以要求电脑支持UEFI，旧电脑不支持。

1.  **安装rEFind**

    rEFInd工具可以安装再windows下，也可以安装在Linux下，本文使用Linux下安装使用，方便快捷。

    ```bash
    非debian系系统自行搜索安装方法
    sudo apt-add-repository ppa:rodsmith/refind  	
    sudo apt-get update  	
    sudo apt-get install refind
    ```

2.  **配置rEFInd**

rEFInd所有的配置信息位于/boot/efi/EFI/refind/refind.conf

其他配置可以默认，更改引导界面背景在refind.conf文件的最后一行，需要加一行指令，这是涉及到refind的主题美化的方面，指令内容为：

```bash
include <相对路径>/theme.config 
```

（相对路径是指你的主题配置文件theme.config在以refind目录为根目录的文件路径）

>   其他配置可以参考文末引用的文章中的信息

<br />

#### 四、制作Linux系统镜像做备份

由于我目前主用Ubuntu18，所以暂时只使用了对Ubuntu的镜像备份方法。

>   SystemBack软件在2016年停止了更新，官方没有后续针对Ubuntu 18.04的软件版本，但其16.04版本的软件在Ubuntu 18.04上仍可以继续使用。这两种操作系统的安装方法稍有不同，如下：

-   **安装SystemBack备份软件**

```bash
#ubuntu 14.04/16.04
sudo add-apt-repository ppa:nemh/systemback
sudo apt update
sudo apt install systemback

#ubuntu 18.04
sudo add-apt-repository "deb http://ppa.launchpad.net/nemh/systemback/ubuntu xenial main"
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 382003C2C8B7B4AB813E915B14E4942973C62A1B
sudo apt update
sudo apt install systemback
```

**SystemBack软件界面选择：**

-   选择"创建Live系统"；
-   **勾选左侧的include the user data files（包含用户数据文件），这样自己主文件夹内的文件都会被包含在系统镜像中。**很多相关的程序的配置文件都是保存在主文件夹内的。Working Directory是设置工作目录，程序运行时产生的临时文件都会被保存在这里。所以一定要保证这里有足够的存储空间；
-   点击"创建新的"开始创建；
-   如果生成的系统镜像小于4G，点击convert to ISO 就可以开始转换；如果大于4G，不能直接转存为iso文件，就要使用采用udf文件系统压缩再转存为光盘文件。

-   **找到systemback生成的文件，以.sblive结尾**

```bash
#解压sblive文件
mkdir sblive
tar -xf /home/systemback_live_2018-10-15.sblive -C sblive

#重命名syslinux为isolinux
mv sblive/syslinux/syslinux.cfg sblive/syslinux/isolinux.cfg
mv sblive/syslinux sblive/isolinux

#安装cdrtools
wget https://nchc.dl.sourceforge.net/project/cdrtools/alpha/cdrtools-3.02a07.tar.gz
tar -xvf cdrtools-3.02a07.tar.gz
cd cdrtools-3.02
make
sudo make install

#生成iso
/opt/schily/bin/mkisofs -iso-level 3 -r -V sblive -cache-inodes -J -l -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -c isolinux/boot.cat -o sblive.iso sblive
```

-   **制作完iso文件后，安装过程和上述LTG安装方式相同，这样在新设备上就可以直接安装自己调教好的系统啦！**

<br />

# 最后

参考文章：

[Windows To Go安装使用手记](https://zhuanlan.zhihu.com/p/59194720)

[在移动硬盘上安装win to go(Windows 10)和Linux to go(ubantu)双系统](https://blog.csdn.net/Pz_Zero/article/details/100044249)

[Windows To Go 辅助工具 WTG辅助工具 v5.5.6](https://bbs.luobotou.org/thread-761-1-1.html)

[rEFInd引导使用教程](https://zhuanlan.zhihu.com/p/67114559)

[把当前ubuntu系统做成镜像](https://www.cnblogs.com/linuxAndMcu/p/10774020.html)

[如何制作ubuntu系统镜像](https://zhuanlan.zhihu.com/p/90022299)

<br />

### 声明

本文仅作为参考文章基础上的总结与个人使用记录，如有侵权，请告知删除。