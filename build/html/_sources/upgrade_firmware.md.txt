# 升级固件

## 前言

本文介绍了如何将主机上的固件文件，通过USB数据线，烧录到开发板的EMMC中。 升级时，需要根据主机操作系统和固件类型来选择合适的升级方式。

## 准备工作

* NXP IMX8M 开发板
* 烧录镜像
* 电脑主机
* 匹配的USB线

### 安装NXP USB驱动

下载[USB驱动](https://www.driverscape.com/download/hid-compliant-vendor-defined-device)然后进行USB驱动安装

### 连接设备

根据以下方法进入升级模式
* 将sw切换到烧录模式
* 通过usb先连接电脑和开发板
* 开发板上电

主机应该会提示发现新硬件并配置驱动。打开设备管理器，会见到新设备出现，如下图。如果没有，则需要返回上一步重新安装驱动。

![winusb 图标](_images/win_usb.png)

### 烧写固件

根据以下步骤开始烧录
* 从网上获取[镜像地址](https://mega.nz/file/135ngAZZ#O9gxgD6Rx5_-ok0dx3QkGLtpZ_COEVsCY0FibkTcRUo
)或者自行编译镜像
* 解压镜像包
* 双击imx-yocto-flash.bat开始烧录，

烧录进程如下图。

![winflash 图标](_images/win_flash.png)

烧录完成后切换为启动模式重新上电开机。

## 常见问题

* 无法进入烧录模式
* 烧录报错
* 无法开机