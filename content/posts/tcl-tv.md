---
title: 'TCL电视开启adb调试'
categories:
    - tech
tags: 
    - TCL
    - TV
date: '2024-08-28 14:34:00'
draft: false
summary: "TCL 50T8H电视的第三方应用安装说明。目前偿试仅能通过adb和u盘安装，无法支持通过电视上的第三方安装软件进行远程安装。"
---

TCL 50T8H电视的第三方应用安装说明。目前偿试仅能通过adb和u盘安装，无法支持通过电视上的第三方安装软件进行远程安装。

## adb设置及连接 
### 开启adb调试

设置 ->系统信息 -> 上下左右

### 查看ip

设置 ->网络 -> 无线网络-> 网络详情

### 安装adb

android-platform-tools

### IP连接设备

```adb connect ip:5555 例如：192.168.1.50:5555```

如果连接成功 使用adb devices查看.需要在电视上确认下允许adb调试连接。

### 开启安装应用权限
```
adb shell setprop persist.tcl.installapk.enable 1
adb shell setprop persist.tcl.debug.installapk 1
```

## 安装应用
```adb install  xxxx.apk```

## 删除应用

### 进入shell
```adb shell```

### 查看已安装的应用
```
pm list package
```

### 查看属于tcl的应用
```
pm list packages | grep 'tcl'
```

### 删除指定应用
例如 T商店
```
pm uninstall --user 0 com.tcl.tshop
```
