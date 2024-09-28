---
title: '小米平板5 Pro开发版切稳定版'
categories:
    - Tech
tags: 
    - 小米
    - 刷机
date: '2024-09-27 10:13:29'
draft: false
summary: "小米平板5 Pro开发版切换到稳定版，并且不需要清除数据的操作记录"
---

我的平板之前通过替换包方法不清资料切换到了开发版（根据以前的经验，停更后开发版和稳定版一般可以互刷不用清数据），现在开发版已停更，想切换到稳定版并更新到Hyper OS系统。之前系统已解锁，并刷了magisk。最新hyper os解锁还要答题，比较麻烦，因此先切换到 MIUI14稳定版，然后通过系统更新直接升级到hyper os。

## 工具准备
### MiFlash 线刷工具
相较于小米助手的刷机功能，线刷还是偏好使用 MiFlash。特点是界面简单纯粹，有自定义高级选项，可以选择刷机不上 BL 锁（对刷 MIUI 海外版来说是必须），自定义刷机脚本，EDL 刷机模式等。MiFlash 也是由小米发布的刷机工具。

```
文件名称：MiFlash2020-3-14-0.rar（最新版本）
文件大小：66.7 MB
发布时间：2020-03-23 16:20:21
MD5指纹：ba6bf711e8647bf9975ad23137690083
更新内容：更新 Fastboot 版本，更新 MES 流程，支持 MTK 全擦刷机流程
```
下载地址：[MiFlash](https://bkt-sgp-miui-ota-update-alisgp.oss-ap-southeast-1.aliyuncs.com/micomm/MiFlash2020-3-14-0.rar)

[MiFlash官网链接](https://miuiver.com/miflash/)

### 固件下载
小米平板 5 Pro WiFi版固件下载，因需要开发版切稳定版，因此只能通过线刷。

[MIUI 14.0.5 线刷包](https://bkt-sgp-miui-ota-update-alisgp.oss-ap-southeast-1.aliyuncs.com/V14.0.5.0.TKYCNXM/elish_images_V14.0.5.0.TKYCNXM_20230920.0000.00_13.0_cn_chinatelecom_4e4fbbaf24.tgz)

[小米平板5系列刷机包官网地址](https://web.vip.miui.com/page/info/mio/mio/detail?postId=32286531)

<b>小米固件下载慢</b>

小米原有链接是 https://bigota.d.miui.com 域名的。
但下载速度只有100kb左右。
可以替换原下载链接中域名为以下域名。

```
cdn-ota.azureedge.net
cdnorg.d.miui.com
bn.d.miui.com
bkt-sgp-miui-ota-update-alisgp.oss-ap-southeast-1.aliyuncs.com
```

## MiFlash刷机

### 对boot.img进行修补
下载固件后，解压出来,将解压文件夹中的 boot.img 文件复制出来。
发到平板上，通过magisk进行修补，然后传回电脑备用。

### 刷入固件
打开MiFlash，选择解压出来的文件夹，右下角选择。
重启平板，黑屏后按住 电源键 和 音量-，直到屏幕显示fastboot，松开。

右下角选择“保留用户数据”(如看不见，可偿试全屏刷机工具窗口)，然后开始刷机，等待进度完成，预计5-10分钟。完成后会自动重启。

![刷机界面](miflash.webp)

```
注意：Miflash线刷需要先将手机解BL锁
```

### 刷入boot.img
刷完之后，重启再进fastboot，刷入修补后的boot.img

```
#查看当前boot slot
fastboot getvar current-slot

#刷入对应的 slot
fastboot flash boot_a 

fastboot reboot
```

### 更新到HyperOS
进入系统后，再通过系统更新升级到HyperOS，更新完后在重启之前，直接用magisk刷入未使用的slot，重启即OK。