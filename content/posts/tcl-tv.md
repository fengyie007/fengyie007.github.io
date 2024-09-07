---
title: 'TCL电视开启adb调试'
categories:
    - tech
tags: 
    - TCL
    - TV
date: '2024-08-28 14:34:00'
draft: false
---

# 一、adb设置及连接 
## 1.开启adb调试

设置 ->系统信息 -> 上下左右

## 2.查看ip

设置 ->网络 -> 无线网络-> 网络详情

## 3.安装adb

android-platform-tools

## 4.IP连接设备

```adb connect ip:5555 例如：192.168.1.50:5555```

如果连接成功 使用adb devices查看.需要在电视上确认下允许adb调试连接。

## 5.开启安装应用权限
```
adb shell setprop persist.tcl.installapk.enable 1
adb shell setprop persist.tcl.debug.installapk 1
```

# 二、安装应用
```adb install  xxxx.apk```

# 三、删除应用

## 1.进入shell
```adb shell```

## 2.查看已安装的应用
```
pm list package
```

## 3.查看属于tcl的应用
```
pm list packages | grep 'tcl'
```

## 4.删除指定应用
例如 T商店
```
pm uninstall --user 0 com.tcl.tshop
```
