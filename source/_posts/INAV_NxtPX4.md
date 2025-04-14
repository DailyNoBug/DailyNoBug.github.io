---
title: INAV_NxtPx4
date: 2025-04-14 17:12:21
tags: 硬件,四轴


---

# INAV_NxtPx4

为基于STM32H743VIH6主控的新飞控板适配INAV固件，需要结合硬件设计与INAV固件的开发框架进行定制化配置。以下是适配的核心步骤及注意事项：

---

### **一、开发环境搭建**
1. **获取INAV源码**  
   从GitHub克隆INAV仓库，切换到支持H7系列的最新分支（如INAV 8.0及以上版本）。  
   
   ```bash
   git clone https://github.com/INAVFlight/inav.git
   ```
   
2. **配置工具链**  
   安装ARM GCC编译工具链（如`gcc-arm-none-eabi`），并确保CMake版本兼容。H743需支持STM32H7系列的编译配置。

---

### **二、硬件描述文件配置**
INAV通过`target`目录下的硬件描述文件定义飞控板的资源分配，需基于现有类似型号（如MATEKF405或官方支持的H743型号）进行修改：  
1. **创建新目标目录**  
   在`src/main/target`下新建目录（如`CUSTOM_H743`），复制相近型号的配置文件（如`target.h`、`target.c`、`CMakeLists.txt`）。

2. **主控芯片定义**  
   在`CMakeLists.txt`中指定主控型号，例如：  
   ```cmake
   target_stm32h743xx(CUSTOM_H743)
   ```

3. **引脚与外设映射**  
   - **LED与蜂鸣器**：定义状态指示灯和蜂鸣器引脚。  
     ```c
     #define LED0 PC13
     #define BEEPER PB8
     ```
   - **SPI/I2C总线**：配置传感器接口。例如双IMU（如BMI270、ICM-42688）需指定SPI总线及片选引脚：  
     ```c
     #define USE_SPI_DEVICE_1
     #define SPI1_SCK_PIN PA5
     #define BMI270_CS_PIN PC2
     ```
   - **UART接口**：映射串口功能（GPS、接收机、图传等）。例如：  
     ```c
     #define USE_UART3
     #define UART3_RX_PIN PC11  // GPS接口
     ```

4. **传感器配置**  
   - **陀螺仪与加速度计**：启用并指定型号及校准方向（如`IMU_BMI270_ALIGN CW0_DEG`）。  
   - **气压计与磁力计**：若使用DPS310或IST8310，需启用对应驱动并指定I2C总线。  
   - **OSD与TF卡**：配置AT7456E OSD芯片及SD卡接口（若支持）。

---

### **三、固件编译与烧录**
1. **编译命令**  
   使用CMake生成编译配置，指定目标名称：  
   ```bash
   make CUSTOM_H743
   ```

2. **烧录固件**  
   - **DFU模式**：按住BOOT按钮连接USB，使用地面站（如INAV Configurator）或`dfu-util`工具烧录。  
   - **SWD调试**：通过ST-Link或J-Link直接烧录（需配置`USE_SWD`）。

---

### **四、地面站调试与验证**
1. **连接与初始化**  
   使用INAV Configurator连接飞控，检查传感器数据（陀螺仪、加速度计、气压计）是否正常显示。

2. **功能配置**  
   - **接收机协议**：在CLI中设置`serialrx_provider`（如SBUS、CRSF）。  
   - **电机输出**：配置PWM/DShot协议及电机映射顺序。  
   - **OSD叠加**：启用并调整显示内容（如电压、飞行模式）。

3. **校准与测试**  
   - **传感器校准**：执行加速度计、陀螺仪、磁力计校准。  
   - **电机测试**：通过地面站逐步测试各电机响应，确保无冲突。

---

### **五、常见问题与优化**
1. **驱动缺失**  
   若电脑无法识别飞控，需安装STM32 VCP驱动（如`stm32cubeprog`包含的驱动）。

2. **硬件冲突**  
   - **电源干扰**：确保传感器（如IMU）独立供电以减少噪声。  
   - **引脚复用**：避免UART与SPI/I2C引脚冲突（参考芯片数据手册）。

3. **性能优化**  
   - **DMA配置**：为高频外设（如陀螺仪SPI）启用DMA以降低CPU负载。  
   - **时钟分配**：优化STM32H7的时钟树，确保外设时钟与主频匹配。

---

### **参考案例**
- **MicoAir743飞控**：支持INAV 8.0+，其硬件描述文件可直接参考（如UART映射、传感器配置）。  
- **星火计划H743飞控**：采用双IMU设计，需在`target.h`中分别定义两个陀螺仪的SPI总线及校准参数。

通过以上步骤，结合硬件特性调整配置文件，即可完成INAV对STM32H743VIH6飞控板的适配。开发过程中建议参考官方文档及社区案例以解决具体问题。