---
title: MFly-Hardware
date: 2025-02-19 10:12:21
tags: 硬件,四轴

---

# 飞行器硬件设计

本文主要记录飞行器硬件设计的过程,以及一些问题记录.

本飞行器旨在作为一个低成本的实验平台完成相关算法和工程落地实践

## 飞行器硬件拓扑

### 飞机整体硬件拓扑

<img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/%E9%A3%9E%E8%A1%8C%E5%99%A8%E7%BB%93%E6%9E%84%E5%9B%BE.drawio.png" alt="img" style="zoom: 80%;" />

### 飞控硬件拓扑

<img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/4-1%E9%A3%9E%E6%8E%A7%E7%BB%93%E6%9E%84%E5%9B%BE.drawio.png" alt="img" style="zoom:80%;" />

### 遥控器硬件拓扑

<img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/%E9%81%A5%E6%8E%A7%E5%99%A8%E7%BB%93%E6%9E%84.drawio.png" alt="img" style="zoom:80%;" />

## 飞控SOC域硬件设计

### 核心板部分

核心板部分是使用som-rk3399核心板,如图所示:

<img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/image-20250219155216587.png" alt="image-20250219155216587" style="zoom: 50%;" />

<center><p>SOM-RK3399核心板俯视图<p/><center/>

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

<center><p>SOM-RK3399硬件引脚图<p/><center/>

### HDMI电路部分

核心板中引出了HDMI接口的引脚,我们需要在底板上进行实现,其中HDMI引脚电路如图所示:

<center class="half">
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/20250219162624.png" width="300"/>
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/20250219162719.png" width="300"/>
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/20250219162747.png" width="300"/>
</center>

<center>
    <p>
        底板HDMI原理图
    </p>
</center>

### SD卡电路部分

核心板引出了`SDMMC0`相关引脚,这部分引脚可以画SD卡模块的电路,下面是这部分的原理图:

<center class="half">
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/image-20250219163109553.png" alt="image-20250219163109553" width="300"/>
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/20250219163351.png" width="300"/>
</center>

<center>
    <p>
        底板SD卡模块原理图
    </p>
</center>

### RJ45网口模块

核心板上有一个PHY芯片,引出以太网引脚,下方为RJ45模块原理图:

<center>
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/20250219164043.png" width="300">
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/20250219164124.png" width="300">
</center>

<center>
    <p>
        RJ45模块原理图
    </p>
</center>

### USB网卡电路

USB网卡使用的是`BL-M8812EU2`,打算使用这个芯片作为图传芯片使用,原理图如下,封装使用的是自己画的封装,存在部分小瑕疵,但是并不影响使用:

<center>
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/20250219164653.png" width="300">
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/20250219164755.png" width="300">
    <img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/20250219164854.png" width="300">
</center>

<center>
    <p>USB网卡模块原理图</p>
</center>


## 电源方案汇总

**注意电阻选择上，有些数值在绘制的时候需要使用叠加来凑数**

### 12V -> 5V方案

使用**MP2236GJ-Z**芯片 [datasheet](https://www.monolithicpower.com/en/documentview/productdocument/index/version/2/document_type/Datasheet/lang/en/sku/MP2236)

主要特点和典型电路如下:

<img src="https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/image-20250219162207918.png" alt="image-20250219162207918" style="zoom: 67%;" />

支持3V-18V输入,6A输出,对于此方案下,12V输入,5V输出的要求,绘制原理图如下:

![](https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/image-20250219162328789.png)

### 12V -> 3.3V方案

依旧使用**MP2236GJ-Z**芯片 [datasheet](https://www.monolithicpower.com/en/documentview/productdocument/index/version/2/document_type/Datasheet/lang/en/sku/MP2236)设计方案同上，但是在buck电阻上存在数据差别

## 飞控传感器部分

### 气压计ICP-20100

datasheet中的典型电路：

![image-20250226201805401](https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/image-20250226201805401.png)

## 飞控外设分配

![image-20250227202626796](https://tuchuang-e682.obs.cn-north-1.myhuaweicloud.com/image-20250227202626796.png)

## 参考文档

1. [SOM-RK3399文档](https://wiki.friendlyelec.com/wiki/index.php/SOM-RK3399/zh)
2. [MP2236GJ-Z datasheet](https://www.monolithicpower.com/en/documentview/productdocument/index/version/2/document_type/Datasheet/lang/en/sku/MP2236)
3. 

## 附录1 芯片选择部分

芯片选型原则：

1. 有完整的芯片数据手册，有参考设计方案
2. 最好有相应的封装库进行设计，但是如果使用lceda进行设计需要审核一下使用的封装是否正确
3. 选择可以在电商平台买到的芯片进行设计

## 附录2 打板前CheckList

| 检查项                     | 是否完成 | 备注 |
| -------------------------- | -------- | ---- |
| 电源电阻是否已经完成凑数   | - [ ]    |      |
| 图传网卡需要独立出一块小板 | - [ ]    |      |
| MIPI屏幕在第一版上暂时取消 | - [ ]    |      |

