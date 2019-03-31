---
title: 蜗牛星际NAS安装黑裙+洗白
date: 2019-03-31 16:04:53
tags: 黑裙、蜗牛星际
---


感谢3月份的“矿难”，就上了蜗牛星际的车，在咸鱼上收了一台。

> 基本配置

4盘位 + J1900处理器 + 4G RAM + 16G MSATA SSD + 200w电源 + 单口网卡

这是蜗牛星际的C款(单口)，也有双口，就是贵了一点，有条件的可以直接上C双。拿到手就是用U盘做引导安装黑群晖。

## 准备工作

> 硬件准备

* U盘 X 1(我使用的是8G，能装下PE和引导文件);
* 显示器 X 1 和 HDML连接线 X 1
* 鼠标 X 1
* 键盘 X 1(安装的时候，家里没有也能搞)
* 电源线(买的主机没有电源线，一般来说家里的电饭煲的电源线就行，可以线下五金店购买)

> 软件准备

* 黑群晖工具获取器 V1.0(下载引导文件与对应版本系统)
* DiskImg(将引导文件镜像写入内置SSD)
* DiskGenius(分区管理工具，PE版)
* Notepad++(文本编辑器)
* 微PE工具箱
* Mac系统安装Windows虚拟机(这里不多介绍)

打包下载：[百度网盘](https://pan.baidu.com/s/1-8g1d9MbX34itX75rwV69g); 提取码: bvpf

## 制作PE盘

1. 将U盘插入电脑，运行PE盘制作工具，右下角选择“安装PE到U盘”；
![安装PE到U盘](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/PE-1.jpeg)

2. 如图设置，点击“立即安装进U盘”；
![安装PE到U盘](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/PE-2.png)

3. PE盘制作工具生成了两个分区，EFI为PE引导分区，“微PE工具箱”为文件区，将黑群晖引导镜像（img）、DiskImg、DiskGenius（PE版）放入U盘内。
![PE分区](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/PE-3.png)


## 黑裙洗白

这里介绍一种洗白方案，需要花钱，淘宝自行购买。不想洗白的可以跳过！！！

在windows下操作:

1. 安装OSFMount，完成后运行OSFMount (下载途径)[https://pan.baidu.com/s/1Mo_lXEGI1qo-8tUHDgGqDg] 提取码:6eyz；

2. 点击左下角-Mount new，选择下载的img镜像(img镜像目录必须是英文或着数字，不能是中文)；
![选择镜像](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/OSFMount1.png)

3. 选择Partition 0，点击OK；
![选项](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/OSFMount2.png)

4. 再把下面Read-only drive的选项去勾后点击OK；
![选项](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/OSFMount3.png)

5. 在OSFMount软件里双击镜像后打开grub目录，编辑grub.cfg文件，建议使用Notepad++编辑grub.cfg文件
![查看grub.cfg文件](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/OSFMount4.png)

6. 根据自己的要求填写正版的SN码和Mac地址，就可以全洗白了
![填写SN码和MAC地址](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/OSFMount6.jpeg)

7. 修改完成后点击Dismount all & Exit
![退出](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/OSFMount5.png)



## 引导文件写入NAS机SSD

1. 将制作好的PE U盘插入蜗牛星际背部的USB接口，通电开机，有键盘的话，可以按下F7选择PE U盘启动(没有键盘也能操作的);

2. 使用DiskGenius，将自带16G SSD内所有分区删除并保存;

3. 启动 DiskImg，驱动器选择机器内置SSD，浏览选择镜像写入(路径、文件名不能有任何中文字符);
![启动 DiskImg](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/SSD-3.png)

4. 写入完成后，使用DiskGenius(PE自带版本太旧了，一定要用复制进去的DiskGenius)，浏览系统内置SSD引导分区文件(ESP)，将grub.cfg文件复制到U盘(之后洗白要用)；

5. 设备关机，拔出U盘。


## 安装DSM到NAS机

1. NAS主机连接网线并启动(此时显示器上显示Happy Hacking)；

2. Mac上安装Synology Assistant(群晖助手)，保持在同一局域网并搜索，搜出结果后，最后选择联机；
![群晖助手](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/DSM-2.png)

3. 选中NAS主机安装DSM系统，浏览找到pat格式，依照提示安装即可；

4. 这里不要把自动更新打开，也不要开启任何反馈计划；
![安装DSM选项](https://raw.githubusercontent.com/dyike/dyike.github.io/master/images/nas/DSM-4.jpeg)

5. 初始化DSM完成，进入桌面后，关机。


## 最后

如果上面的步骤，你有全洗白的话，到此应该是安装黑裙成功，并洗白了。开机体验，最大的缺点就是噪音挺大的。

1. NAS主机(249元) + 河南->上海顺丰运费(46元)

2. 一块西部数据2T红盘(330元，咸鱼收)

3. 黑裙洗白(30元)

4. 更换了电源风扇(12元)



























