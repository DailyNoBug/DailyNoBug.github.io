---
title: MFly-Hardware
date: 2025-02-19 10:12:21
tags:

---

# 飞行器硬件设计

本文主要记录飞行器硬件设计的过程,以及一些问题记录.

本飞行器旨在作为一个低成本的实验平台完成相关算法和工程落地实践

## 飞行器硬件拓扑

### 飞机整体硬件拓扑

![img](https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/%E9%A3%9E%E8%A1%8C%E5%99%A8%E7%BB%93%E6%9E%84%E5%9B%BE.drawio.png)

### 飞控硬件拓扑

![img](https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/4-1%E9%A3%9E%E6%8E%A7%E7%BB%93%E6%9E%84%E5%9B%BE.drawio.png)

### 遥控器硬件拓扑

![img](https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/%E9%81%A5%E6%8E%A7%E5%99%A8%E7%BB%93%E6%9E%84.drawio.png)

## 飞控SOC域硬件设计

### 核心板部分

核心板部分是使用som-rk3399核心板,如图所示:

![image-20250219155216587](https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/image-20250219155216587.png)

- SOM-RK3399是友善电子团队设计的一款266-pin金手指形式高性能ARM计算机模块，它采用了瑞芯微64位六核SoC RK3399作为主处理器，标配2GB DDR3内存和16GB闪存，板载2x2 MIMO双天线WiFi模组，尺寸只有69.6x50mm，模块上带有独立的TypeC供电接口，以及USB-C显示接口，无需底板也可以单独使用。

- SOM-RK3399计算模块具有丰富的外设和扩展接口，通过底板可连接使用4通道NVMe高速固态硬盘，读写速度高达1GB/s; 它还可以扩展使用双MIPI宽动态摄像头，另外它还带有eDP显示接口，MIPI显示接口, 1路USB3.0, 2路USB2.0, 以及I2C, I2S, SPI, PWM, GPIO和串口等各种资源。

相关核心板硬件部分可以参考链接:https://wiki.friendlyelec.com/wiki/index.php/SOM-RK3399/zh

该核心板使用0.5mm Pitch 260-Pin **Standard Type** DDR4 SODIMM Socket卡槽,

参考型号为:https://www.te.com/usa-en/product-2309409-5.html

根据卡槽引脚定义可以绘制相关原理图:

<center class="half">
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/image-20250219155533770.png" width="300"/>
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/image-20250219155554870.png" width="300"/>
</center>

### 电源方案汇总



