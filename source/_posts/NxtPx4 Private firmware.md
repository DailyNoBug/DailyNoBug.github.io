---
title: NxtPx4 Private firmware
date: 2025-04-15 10:11:51
tags:

---

# NxtPx4 Private firmware

## SPI For BMI088

1. **启动 CubeMX 并选择芯片:** 打开 STM32CubeMX，创建一个新项目，选择你的目标芯片 `STM32H743VIHx`。
2. **配置 SPI 外设:**
   - 在左侧的 "Pinout & Configuration" 标签页中，展开 "Connectivity" 类别。
   - 选择一个 SPI 外设 (例如 `SPI1` 或 `SPI2`)。
   - 设置 **Mode** 为 `Full-Duplex Master`。
   - 在 **Configuration** -> **Parameter Settings** 中：
     - **Frame Format:** Motorola
     - **Data Size:** 8 Bits
     - **First Bit:** MSB First
     - **Clock Polarity (CPOL):** Low (根据 BMI088 数据手册，支持 Mode 0 和 Mode 3，我们选择常用的 Mode 0)
     - **Clock Phase (CPHA):** 1 Edge (Mode 0)
     - **NSS Signal Type:** `Disable` (我们将使用软件控制 GPIO 作为片选)。
     - **Baud Rate Prescaler:** 选择一个预分频器，使得最终的 SPI 时钟速率低于或等于 BMI088 支持的最大速率 (10 MHz)。例如，如果 SPI 的时钟源是 100 MHz，选择预分频器 16 会得到 6.25 MHz，这是一个安全的选择。你需要根据你的 "Clock Configuration" 来计算。
3. **配置 GPIO 引脚:**
   - **SPI 引脚:** 在 Pinout 视图中，找到你选择的 SPI 外设对应的 SCK, MISO, MOSI 引脚，将它们的功能设置为对应的 `SPIx_SCK`, `SPIx_MISO`, `SPIx_MOSI` (它们会自动配置为 Alternate Function Push-pull)。
   - **片选 (CS/NSS) 引脚:**
     - 选择**两个**空闲的 GPIO 引脚。一个用于加速度计片选 (例如 `PC4`，命名为 `ACC_CS`)，另一个用于陀螺仪片选 (例如 `PC5`，命名为 `GYR_CS`)。
     - 将这两个引脚配置为 `GPIO_Output`。
     - 在 **Configuration** -> **GPIO Settings** 中：
       - 设置 **GPIO Output level** 为 `High` (SPI 片选通常是低电平有效，所以默认拉高)。
       - 设置 **GPIO mode** 为 `Output Push Pull`。
       - 设置 **Maximum output speed** 为 `High` 或 `Very High`。
       - 可以设置 **User Label** 为 `ACC_CS` 和 `GYR_CS` 以方便识别。
   - **(可选) 中断引脚:** BMI088 有中断输出引脚 (INT1_ACC, INT1_GYR)。如果需要使用中断来读取数据（推荐，避免轮询），选择一或两个 GPIO 引脚配置为 `GPIO_Input`，并启用对应的 `EXTI` 中断模式 (例如 `Rising Edge trigger detection`)。在 **NVIC Settings** 中启用对应的 EXTI 中断。
4. **时钟配置 (Clock Configuration):**
   - 检查 "Clock Configuration" 标签页，确保你选择的 SPI 外设的时钟源 (例如 APB1/APB2) 已启用，并且频率设置正确，以便计算波特率预分频器。H7 系列时钟配置较复杂，请仔细检查。
5. **项目管理 (Project Manager):**
   - 设置项目名称和路径。
   - 选择你的 Toolchain / IDE (例如 MDK-ARM, STM32CubeIDE)。
   - 在 **Code Generator** 标签页，勾选 "Generate peripheral initialization as a pair of '.c/.h' files per peripheral"。
6. **生成代码:** 点击 "Generate Code"。

## USB虚拟串口部分

### STM32CubeMX 配置步骤

1. **打开 CubeMX 工程:** 打开你的 STM32H743VIHx 项目。
2. **启用 USB OTG FS:**
   - 在左侧 "Pinout & Configuration" 标签页 -> "Connectivity" 中，点击 `USB_OTG_FS`。
   - 在 Mode 区域，勾选 `Device_Only`。
   - *(可选但常见)* 激活 `Activate_VBUS`。这通常对应 `PA9` 引脚，用于检测 USB VBUS 电压，确保你的硬件设计支持 VBUS 检测。
3. **启用 USB Device Middleware:**
   - 在左侧 "Pinout & Configuration" 标签页 -> "Middleware" 中，点击 `USB_DEVICE`。
   - 勾选 `Enabled` 复选框。
   - 在右侧出现的 "Class for FS IP" 下拉菜单中，选择 `Communication Device Class (Virtual Port Com)`。
4. **检查 USB Device 参数 (可选):**
   - 在 "Middleware" -> `USB_DEVICE` -> "Configuration" -> "Parameter Settings" 中，你可以查看或修改 VID, PID, 制造商/产品字符串等。默认值通常可以直接使用。注意 `CDC_RX_DATA_SIZE` 和 `CDC_TX_DATA_SIZE` 定义了 VCP 的收发缓冲区大小。
5. **配置 USB 时钟:**
   - **非常重要:** USB FS 需要一个精确的 48 MHz 时钟。转到 "Clock Configuration" 标签页。
   - 你需要确保 USB 的时钟源 (通常来自 HSI48 或通过 PLL 配置外部晶振 HSE) 被正确设置为 48 MHz。如果使用 HSI48，直接选择它作为 USB 时钟源。如果使用 HSE+PLL，需要调整 PLL 设置（通常是 PLL1Q 或 PLL3Q）来产生 48 MHz。**时钟配置不正确是 USB 无法工作的常见原因。**
6. **检查 GPIO:**
   - 启用 `USB_OTG_FS` 后，对应的 `PA11 (DM)` 和 `PA12 (DP)` 引脚应该会自动配置为 USB 的 Alternate Function 模式。确认一下 Pinout 视图中的引脚状态。
7. **生成代码:**
   - 点击 "Project Manager" 设置好项目选项。
   - 点击 "Generate Code"。CubeMX 会生成初始化代码，并添加 USB Device 库文件到你的项目中 (包括 `Middlewares` 目录下的 `ST/STM32_USB_Device_Library` 和 `App` 目录下的 `usb_device.c/h`, `usbd_cdc_if.c/h` 等)。

## 烧录上位机开发

```python
import datetime
import sys
import os
import subprocess
import time
import serial
import serial.tools.list_ports
from PyQt6.QtWidgets import (
    QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout, QGridLayout,
    QTabWidget, QGroupBox, QLabel, QLineEdit, QPushButton, QComboBox,
    QTextEdit, QFileDialog, QMessageBox, QStatusBar, QCheckBox, QSizePolicy
)
from PyQt6.QtCore import QThread, pyqtSignal, QObject, Qt, QTimer, QMetaObject, pyqtSlot
from PyQt6.QtGui import QTextCursor # 用于滚动文本框

# --- 常量 ---
DFU_VID_PID = "0483:df11" # STM32 DFU VID/PID
DEFAULT_ALT_SETTING = 0   # 默认 Alt Setting
DEFAULT_FLASH_ADDRESS = "0x08000000" # 默认烧录地址
DFU_UTIL_COMMAND = "dfu-util" # dfu-util 命令

def get_time_stamp():
    ct = time.time()
    local_time = time.localtime(ct)
    data_head = time.strftime("%Y-%m-%d %H:%M:%S", local_time)
    data_secs = (ct - int(ct)) * 1000
    time_stamp = "%s.%03d" % (data_head, data_secs)
    return time_stamp

# --- 串口读取 Worker (运行在 QThread) ---
class SerialWorker(QObject):
    """处理串口读写的 Worker 对象"""
    data_received = pyqtSignal(bytes)    # 信号：接收到数据
    error_occurred = pyqtSignal(str)     # 信号：发生错误
    finished = pyqtSignal()              # 信号：任务完成

    def __init__(self, serial_instance):
        super().__init__()
        self.ser = serial_instance
        self._is_running = False

    @pyqtSlot()
    def run(self):
        """开始循环读取串口数据"""
        self._is_running = True
        while self._is_running:
            try:
                if self.ser and self.ser.is_open:
                    # 检查是否有数据等待读取
                    if self.ser.in_waiting > 0:
                        data = self.ser.read(self.ser.in_waiting)
                        if data:
                            print(type(data))
                            self.data_received.emit(data) # 发送接收到的数据信号
                    else:
                        # 短暂休眠，避免空转占用过多 CPU
                        QThread.msleep(10) # 使用 QThread.msleep 避免阻塞事件循环
                else:
                    # 如果串口意外关闭，则停止
                    if self._is_running:
                        # Check again to prevent emitting error after explicit stop
                        if self._is_running:
                            self.error_occurred.emit("串口似乎已关闭")
                    self._is_running = False # 退出循环

            except serial.SerialException as e:
                if self._is_running: # 避免在手动停止后再次发送错误
                    self.error_occurred.emit(f"串口读取错误: {e}")
                self._is_running = False # 退出循环
            except Exception as e:
                if self._is_running:
                    self.error_occurred.emit(f"读取线程未知错误: {e}")
                self._is_running = False # 退出循环

        self.ser = None # 清理引用
        self.finished.emit() # 发送完成信号

    @pyqtSlot()
    def stop(self):
        """请求停止读取循环"""
        self._is_running = False

# --- DFU 烧录 Worker (运行在 QThread) ---
class DfuWorker(QObject):
    """处理 DFU 烧录命令的 Worker 对象"""
    progress_update = pyqtSignal(str) # 信号：更新进度文本
    finished = pyqtSignal(int, str)   # 信号：任务完成 (返回码, 最终消息)

    def __init__(self, command_list):
        super().__init__()
        self.command = command_list
        self._is_running = False
        self.process = None # 用于持有 subprocess 实例

    @pyqtSlot()
    def run(self):
        """执行 dfu-util 命令"""
        self._is_running = True
        final_message = ""
        return_code = -1 # 默认错误码

        try:
            self.progress_update.emit(f"执行命令: {' '.join(self.command)}\n")
            # 使用 Popen 以便可以读取实时输出
            self.process = subprocess.Popen(self.command,
                                            stdout=subprocess.PIPE,
                                            stderr=subprocess.STDOUT, # 合并标准错误到标准输出
                                            text=True,
                                            encoding='utf-8', # 指定编码
                                            errors='ignore',  # 忽略解码错误
                                            bufsize=1, # 行缓冲
                                            universal_newlines=True,
                                            creationflags=subprocess.CREATE_NO_WINDOW if sys.platform == 'win32' else 0)

            # 实时读取输出
            if self.process.stdout:
                while self._is_running:
                    line = self.process.stdout.readline()
                    if not line: # 进程结束
                        break
                    # Check running flag again before emitting, in case stop() was called
                    if self._is_running:
                        self.progress_update.emit(line)
                # Check running flag before closing stdout
                if self._is_running:
                    self.process.stdout.close()


            # 等待进程结束并获取返回码 (如果仍在运行)
            if self._is_running and self.process:
                return_code = self.process.wait()
                if return_code == 0:
                    final_message = "\n--- 烧录成功完成 ---\n"
                else:
                    final_message = f"\n--- 烧录失败 (返回码: {return_code}) ---\n"
                self.progress_update.emit(final_message)

            elif not self._is_running: # 如果是手动停止的
                final_message = "\n--- 烧录中止 ---\n"
                return_code = -99 # 自定义中止码
                # Check if process exists before emitting update
                if self.process:
                    self.progress_update.emit(final_message)


        except FileNotFoundError:
            final_message = f"错误: 未找到 '{DFU_UTIL_COMMAND}' 命令。\n请确保已安装并添加到 PATH。\n"
            self.progress_update.emit(final_message)
            return_code = -1
        except Exception as e:
            final_message = f"\n--- 烧录过程中发生异常 ---\nError: {e}\n"
            self.progress_update.emit(final_message)
            return_code = -2
        finally:
            self._is_running = False
            self.process = None # 清理 process 引用
            self.finished.emit(return_code, final_message) # 发送完成信号

    @pyqtSlot()
    def stop(self):
        """请求停止 DFU 进程"""
        self._is_running = False
        if self.process and self.process.poll() is None: # 如果进程仍在运行
            try:
                self.process.terminate() # 尝试终止进程
                # Check if process exists before emitting update
                if self.process:
                    self.progress_update.emit("尝试中止 DFU 进程...\n")
            except Exception as e:
                # Check if process exists before emitting update
                if self.process:
                    self.progress_update.emit(f"中止进程时出错: {e}\n")


# --- 主应用窗口 ---
class STM32ToolAppPyQt(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("STM32 DFU烧录 & 串口助手 (PyQt6)")
        self.setGeometry(100, 100, 850, 650) # x, y, width, height

        # --- 成员变量 ---
        self.firmware_path = ""         # DFU 固件路径
        self.serial_instance = None     # pyserial 实例
        self.serial_thread = None       # 串口读取线程 (QThread)
        self.serial_worker = None       # 串口 Worker (QObject)
        self.dfu_thread = None          # DFU 烧录线程 (QThread)
        self.dfu_worker = None          # DFU Worker (QObject)

        # --- 初始化 UI ---
        self._init_ui()

        # --- 启动检查 ---
        self.check_dfu_util_exists()
        # 使用 QTimer 延迟执行，确保主窗口已显示
        QTimer.singleShot(100, self.refresh_dfu_device)
        QTimer.singleShot(100, self.refresh_com_ports)


    def _init_ui(self):
        """初始化用户界面"""
        # --- 中心控件和主布局 ---
        self.central_widget = QWidget()
        self.setCentralWidget(self.central_widget)
        self.main_layout = QVBoxLayout(self.central_widget)

        # --- 创建选项卡控件 ---
        self.tab_widget = QTabWidget()
        self.main_layout.addWidget(self.tab_widget)

        # --- 创建 DFU 和串口选项卡 ---
        self.dfu_tab = QWidget()
        self.serial_tab = QWidget()
        self.tab_widget.addTab(self.dfu_tab, "DFU 烧录")
        self.tab_widget.addTab(self.serial_tab, "串口助手")

        # --- 布局 DFU 选项卡 ---
        self._setup_dfu_tab()

        # --- 布局串口选项卡 ---
        self._setup_serial_tab()

        # --- 创建状态栏 ---
        self.status_bar = QStatusBar()
        self.setStatusBar(self.status_bar)
        self.dfu_status_label = QLabel("DFU: 空闲")
        self.serial_status_label = QLabel("串口: 已断开")
        self.status_bar.addPermanentWidget(self.dfu_status_label, 1) # 添加伸展因子
        self.status_bar.addPermanentWidget(self.serial_status_label, 1)

    def _setup_dfu_tab(self):
        """设置 DFU 选项卡内的控件和布局"""
        layout = QVBoxLayout(self.dfu_tab)

        # --- DFU 控制组 ---
        control_group = QGroupBox("DFU 控制")
        control_layout = QGridLayout(control_group)
        layout.addWidget(control_group)

        # DFU 设备检测
        self.refresh_dfu_btn = QPushButton("检测 DFU 设备")
        self.refresh_dfu_btn.clicked.connect(self.refresh_dfu_device)
        self.dfu_device_label = QLabel("未检测到 DFU 设备")
        self.dfu_device_label.setWordWrap(True)
        control_layout.addWidget(self.refresh_dfu_btn, 0, 0)
        control_layout.addWidget(self.dfu_device_label, 0, 1, 1, 2) # 跨两列

        # 固件选择
        control_layout.addWidget(QLabel("固件文件 (.bin):"), 1, 0)
        self.firmware_path_edit = QLineEdit()
        self.firmware_path_edit.setReadOnly(True)
        self.browse_firmware_btn = QPushButton("浏览...")
        self.browse_firmware_btn.clicked.connect(self.browse_firmware)
        control_layout.addWidget(self.firmware_path_edit, 1, 1)
        control_layout.addWidget(self.browse_firmware_btn, 1, 2)

        # DFU 参数
        param_layout = QHBoxLayout()
        param_layout.addWidget(QLabel("Alt Setting:"))
        self.alt_setting_edit = QLineEdit(str(DEFAULT_ALT_SETTING))
        self.alt_setting_edit.setFixedWidth(50)
        param_layout.addWidget(self.alt_setting_edit)
        param_layout.addWidget(QLabel("烧录地址:"))
        self.flash_addr_edit = QLineEdit(DEFAULT_FLASH_ADDRESS)
        self.flash_addr_edit.setFixedWidth(120)
        param_layout.addWidget(self.flash_addr_edit)
        param_layout.addStretch() # 添加伸展，使参数靠左
        control_layout.addLayout(param_layout, 2, 0, 1, 3) # 跨三列

        # 烧录按钮
        self.flash_button = QPushButton("开始烧录")
        self.flash_button.clicked.connect(self.flash_firmware)
        control_layout.addWidget(self.flash_button, 3, 0, 1, 3) # 跨三列居中 (默认)

        # --- DFU 输出组 ---
        output_group = QGroupBox("DFU 输出")
        output_layout = QVBoxLayout(output_group)
        layout.addWidget(output_group)

        self.dfu_output_text = QTextEdit()
        self.dfu_output_text.setReadOnly(True)
        self.dfu_output_text.setFontFamily("Courier New") # 等宽字体
        output_layout.addWidget(self.dfu_output_text)

        layout.addStretch() # 添加伸展，使控件在垂直方向上不扩散

    def _setup_serial_tab(self):
        """设置串口选项卡内的控件和布局"""
        layout = QVBoxLayout(self.serial_tab)

        # --- 串口设置组 ---
        settings_group = QGroupBox("串口设置")
        settings_layout = QGridLayout(settings_group)
        layout.addWidget(settings_group)

        # COM 口选择
        settings_layout.addWidget(QLabel("COM 端口:"), 0, 0)
        self.com_port_combo = QComboBox()
        self.refresh_com_btn = QPushButton("刷新")
        self.refresh_com_btn.clicked.connect(self.refresh_com_ports)
        settings_layout.addWidget(self.com_port_combo, 0, 1)
        settings_layout.addWidget(self.refresh_com_btn, 0, 2)

        # 波特率选择
        settings_layout.addWidget(QLabel("波特率:"), 1, 0)
        self.baud_rate_combo = QComboBox()
        baud_rates = ["9600", "19200", "38400", "57600", "115200", "230400", "460800", "921600", "1500000"]
        self.baud_rate_combo.addItems(baud_rates)
        self.baud_rate_combo.setCurrentText("1500000") # 设置默认值
        settings_layout.addWidget(self.baud_rate_combo, 1, 1)

        # 连接/断开按钮
        self.connect_button = QPushButton("打开串口")
        self.connect_button.setCheckable(True) # 设置为可切换状态的按钮
        self.connect_button.toggled.connect(self.toggle_serial_connection)
        settings_layout.addWidget(self.connect_button, 0, 3, 2, 1) # 跨两行

        # --- 接收区组 ---
        receive_group = QGroupBox("接收区")
        receive_layout = QVBoxLayout(receive_group)
        layout.addWidget(receive_group)

        self.receive_text = QTextEdit()
        self.receive_text.setReadOnly(True)
        self.receive_text.setFontFamily("Courier New")
        receive_layout.addWidget(self.receive_text)

        # 接收区底部按钮和选项
        receive_bottom_layout = QHBoxLayout()
        self.hex_display_check = QCheckBox("Hex 显示")
        self.clear_receive_btn = QPushButton("清空接收")
        self.clear_receive_btn.clicked.connect(self.clear_receive_text)
        receive_bottom_layout.addWidget(self.hex_display_check)
        receive_bottom_layout.addStretch()
        receive_bottom_layout.addWidget(self.clear_receive_btn)
        receive_layout.addLayout(receive_bottom_layout)

        # --- 发送区组 ---
        send_group = QGroupBox("发送区")
        send_layout = QHBoxLayout(send_group) # 使用水平布局
        layout.addWidget(send_group)

        self.send_entry = QLineEdit()
        self.send_entry.returnPressed.connect(self.send_serial_data) # 按回车发送
        self.send_button = QPushButton("发送")
        self.send_button.clicked.connect(self.send_serial_data)
        self.send_button.setEnabled(False) # 默认禁用

        self.send_newline_check = QCheckBox("发送新行 (\\r\\n)")
        self.send_newline_check.setChecked(True)
        self.hex_send_check = QCheckBox("Hex 发送")

        send_layout.addWidget(self.send_entry)
        send_layout.addWidget(self.send_newline_check)
        send_layout.addWidget(self.hex_send_check)
        send_layout.addWidget(self.send_button)

        # 设置发送输入框自动扩展
        self.send_entry.setSizePolicy(QSizePolicy.Policy.Expanding, QSizePolicy.Policy.Preferred)

        layout.addStretch() # 添加伸展

    # --- DFU 相关方法 ---
    @pyqtSlot()
    def refresh_dfu_device(self):
        """检测 DFU 设备"""
        self.dfu_status_label.setText("DFU: 正在检测...")
        self.dfu_device_label.setText("正在检测...")
        QApplication.processEvents() # 处理事件，更新UI

        try:
            # 运行 dfu-util -l 命令，捕获输出，指定编码
            process = subprocess.run(
                [DFU_UTIL_COMMAND, "-l"],
                capture_output=True,
                text=True,          # 输出为文本
                check=False,        # 不检查返回码，手动处理
                encoding='utf-8',   # 指定编码
                errors='ignore',    # 忽略解码错误
                creationflags=subprocess.CREATE_NO_WINDOW if sys.platform == 'win32' else 0
            )
            output = process.stdout + "\n" + process.stderr # 合并 stdout 和 stderr

            # 初始状态
            found_device = False
            found_device_str = "未检测到 STM32 DFU 设备"
            dfu_status_str = "DFU: 未检测到设备"

            lines = output.splitlines()
            for i, line in enumerate(lines):
                # 更精确地匹配 VID/PID
                vid_str = f"{DFU_VID_PID.split(':')[0]}" # e.g., "vid=0483"
                pid_str = f"{DFU_VID_PID.split(':')[1]}" # e.g., "pid=df11"
                # print(f"114515 {vid_str} {pid_str}")
                print(f"114516 {vid_str} {line.lower()}")
                if vid_str in line.lower() and pid_str in line.lower():
                    found_device = True
                    device_name = "STM32 Bootloader" # Default name
                    # Try to parse device name if available
                    if "name=" in line:
                        try:
                            start = line.find('name="') + len('name="')
                            end = line.find('"', start)
                            if start > -1 and end > -1:
                                device_name = line[start:end]
                        except Exception: pass # Ignore parsing errors

                    found_device_str = f"找到设备: {device_name} ({DFU_VID_PID})"
                    dfu_status_str = "DFU: 检测到设备"

                    # Find alternate setting info
                    alt_info = []
                    j = i + 1
                    while j < len(lines) and lines[j].strip().startswith("alt="):
                        alt_info.append(lines[j].strip())
                        j += 1
                    if alt_info:
                        found_device_str += f" | Alts: {', '.join(alt_info)}"
                    break # Found the first matching device, stop searching

            # 更新 UI 和日志
            self.dfu_device_label.setText(found_device_str)
            self.dfu_status_label.setText(dfu_status_str)

            # 记录日志，包括完整输出
            if found_device:
                log_output = f"检测到 DFU 设备:\n{output}\n"
            else:
                log_output = f"未在 dfu-util 输出中找到 VID={DFU_VID_PID.split(':')[0]}, PID={DFU_VID_PID.split(':')[1]} 的设备。\n"
                log_output += "请确认:\n"
                log_output += "1. 设备已连接并处于 DFU 模式。\n"
                log_output += "2. (Windows) 已使用 Zadig 等工具安装正确的 WinUSB 或 libusb 驱动。\n"
                log_output += "3. (Linux) 具有访问 USB 设备的权限 (可能需要 udev 规则)。\n"
                log_output += f"\n--- dfu-util -l 输出 ---\n{output}\n-------------------------\n"

            self._append_dfu_text(log_output) # Log the result and full output

        except FileNotFoundError:
            self.dfu_device_label.setText("错误: 未找到 dfu-util")
            self.dfu_status_label.setText("DFU: 错误")
            self._append_dfu_text(f"错误: 未找到 '{DFU_UTIL_COMMAND}' 命令。\n")
            QMessageBox.critical(self, "错误", f"未找到 '{DFU_UTIL_COMMAND}' 命令。\n请确保已安装 dfu-util 并将其添加到系统 PATH 环境变量中。")
        except Exception as e:
            self.dfu_device_label.setText(f"检测出错: {e}")
            self.dfu_status_label.setText("DFU: 检测错误")
            self._append_dfu_text(f"检测 DFU 设备时出错: {e}\n")
            QMessageBox.critical(self, "错误", f"检测 DFU 设备时出错: {e}")

    @pyqtSlot()
    def browse_firmware(self):
        """浏览选择固件文件"""
        initial_dir = os.path.expanduser("~")
        filepath, _ = QFileDialog.getOpenFileName(
            self,
            "选择固件文件",
            initial_dir,
            "Binary files (*.bin);;All files (*.*)"
        )
        if filepath:
            self.firmware_path = filepath
            self.firmware_path_edit.setText(filepath)
            self._append_dfu_text(f"已选择固件: {filepath}\n")

    @pyqtSlot()
    def flash_firmware(self):
        """开始 DFU 烧录过程"""
        if not self.firmware_path:
            QMessageBox.warning(self, "警告", "请先选择固件文件！")
            return
        if not os.path.exists(self.firmware_path):
            QMessageBox.critical(self, "错误", f"固件文件不存在: {self.firmware_path}")
            return

        if "未检测到" in self.dfu_device_label.text() or "错误" in self.dfu_device_label.text():
            QMessageBox.warning(self, "警告", "未检测到有效的 DFU 设备，请先检测设备。")
            return

        # 检查是否已在烧录中
        if self.dfu_thread and self.dfu_thread.isRunning():
            QMessageBox.warning(self, "警告", "已有一个烧录任务正在进行中。")
            return

        alt_setting = self.alt_setting_edit.text()
        flash_address = self.flash_addr_edit.text()

        # 验证参数
        if not alt_setting.isdigit():
            QMessageBox.critical(self, "错误", "Alt Setting 必须是一个数字。")
            return
        if not flash_address.startswith("0x") or not all(c in '0123456789abcdefABCDEF' for c in flash_address[2:]):
            QMessageBox.critical(self, "错误", "烧录地址格式无效 (应为 0x 开头的十六进制数)。")
            return

        # 准备 DFU 命令
        command = [
            DFU_UTIL_COMMAND,
            "-a", alt_setting,
            "-s", f"{flash_address}:leave", # 添加 :leave 使设备烧录后自动复位
            "-D", self.firmware_path
        ]

        # 更新 UI 状态
        self.flash_button.setEnabled(False)
        self.flash_button.setText("正在烧录...")
        self.dfu_status_label.setText("DFU: 正在烧录...")
        self._append_dfu_text(f"--- 开始烧录: {os.path.basename(self.firmware_path)} ---\n")
        self._append_dfu_text(f"Alt Setting: {alt_setting}, Address: {flash_address}\n")

        # 创建 Worker 和 Thread
        self.dfu_worker = DfuWorker(command)
        self.dfu_thread = QThread()
        self.dfu_worker.moveToThread(self.dfu_thread)

        # 连接信号和槽
        self.dfu_worker.progress_update.connect(self._append_dfu_text)
        self.dfu_worker.finished.connect(self.dfu_finished)
        self.dfu_thread.started.connect(self.dfu_worker.run) # 线程启动后执行 run
        # 清理工作：线程结束后删除 Worker 和 Thread 对象
        self.dfu_worker.finished.connect(self.dfu_thread.quit)
        self.dfu_worker.finished.connect(self.dfu_worker.deleteLater)
        self.dfu_thread.finished.connect(self.dfu_thread.deleteLater)
        self.dfu_thread.finished.connect(self._reset_dfu_state) # 线程结束后重置状态

        # 启动线程
        self.dfu_thread.start()

    @pyqtSlot(int, str)
    def dfu_finished(self, return_code, message):
        """DFU 烧录完成后的处理"""
        # Extract the last meaningful line for the status bar
        last_line = message.strip().splitlines()[-1] if message.strip() else "任务结束"
        self.dfu_status_label.setText(f"DFU: {last_line}")

        if return_code == 0:
            QMessageBox.information(self, "成功", "固件烧录成功！设备将尝试复位。")
        elif return_code != -99: # -99 是用户中止，不弹窗
            QMessageBox.critical(self, "失败", f"固件烧录失败！请检查 DFU 输出信息。\n返回码: {return_code}")

        # 尝试刷新 DFU 设备状态 (可能已退出 DFU 模式)
        QTimer.singleShot(1000, self.refresh_dfu_device) # 延迟执行

    def _reset_dfu_state(self):
        """重置 DFU 相关控件状态"""
        self.flash_button.setEnabled(True)
        self.flash_button.setText("开始烧录")
        self.dfu_thread = None # 清理线程引用
        self.dfu_worker = None

    def _append_dfu_text(self, text):
        """向 DFU 输出文本框追加文本"""
        self.dfu_output_text.moveCursor(QTextCursor.MoveOperation.End)
        self.dfu_output_text.insertPlainText(text)
        self.dfu_output_text.moveCursor(QTextCursor.MoveOperation.End) # 确保滚动到底部

    # --- 串口相关方法 ---
    @pyqtSlot()
    def refresh_com_ports(self):
        """刷新可用 COM 端口列表"""
        current_port = self.com_port_combo.currentText() # Save current selection
        self.com_port_combo.clear() # 清空列表
        ports = serial.tools.list_ports.comports()
        port_names = [port.device for port in ports]
        self.com_port_combo.addItems(port_names)
        # Try to restore previous selection
        if current_port in port_names:
            self.com_port_combo.setCurrentText(current_port)
        elif port_names:
            self.com_port_combo.setCurrentIndex(0) # 默认选中第一个
        self._log_serial_info("已刷新 COM 端口列表。\n")

    @pyqtSlot(bool)
    def toggle_serial_connection(self, checked):
        """根据按钮状态连接或断开串口"""
        if checked: # 按钮被按下，表示要连接
            self._connect_serial()
        else: # 按钮弹起，表示要断开
            self._disconnect_serial()

    def _connect_serial(self):
        """连接到选定的串口"""
        port = self.com_port_combo.currentText()
        baud_str = self.baud_rate_combo.currentText()

        if not port:
            QMessageBox.warning(self, "警告", "请选择一个 COM 端口！")
            self.connect_button.setChecked(False) # 恢复按钮状态
            return
        if not baud_str.isdigit():
            QMessageBox.critical(self, "错误", "波特率必须是一个数字！")
            self.connect_button.setChecked(False)
            return
        baud = int(baud_str)

        try:
            # Attempt to open serial port
            self.serial_instance = serial.Serial()
            self.serial_instance.port = port
            self.serial_instance.baudrate = baud
            self.serial_instance.timeout = 0.1 # Use a small timeout
            self.serial_instance.open()

            if self.serial_instance.is_open:
                self.serial_status_label.setText(f"串口: 已连接 {port} @ {baud}bps")
                self._log_serial_info(f"串口 {port} 已打开，波特率 {baud}。\n")
                self.connect_button.setText("关闭串口") # 更新按钮文本

                # 禁用设置控件
                self.com_port_combo.setEnabled(False)
                self.baud_rate_combo.setEnabled(False)
                self.refresh_com_btn.setEnabled(False)
                self.send_button.setEnabled(True) # 启用发送按钮

                # 创建并启动串口读取 Worker 和 Thread
                self.serial_worker = SerialWorker(self.serial_instance)
                self.serial_thread = QThread(self) # Pass parent for potential better management
                self.serial_worker.moveToThread(self.serial_thread)

                # 连接信号
                self.serial_worker.data_received.connect(self.handle_serial_data)
                self.serial_worker.error_occurred.connect(self.handle_serial_error)
                self.serial_worker.finished.connect(self.serial_worker_finished) # Worker 完成时
                self.serial_thread.started.connect(self.serial_worker.run)
                # 线程结束后清理
                self.serial_thread.finished.connect(self.serial_worker.deleteLater) # Request deletion
                self.serial_thread.finished.connect(self.serial_thread.deleteLater) # Request deletion


                self.serial_thread.start()

            else:
                # This case should theoretically not be reached if open() fails
                QMessageBox.critical(self, "错误", f"无法打开串口 {port}！(is_open is False)")
                self.connect_button.setChecked(False) # 恢复按钮状态

        except serial.SerialException as e:
            QMessageBox.critical(self, "串口错误", f"无法打开串口 {port}:\n{e}")
            self.serial_status_label.setText(f"串口: 打开失败")
            self.serial_instance = None
            self.connect_button.setChecked(False)
        except ValueError:
            QMessageBox.critical(self, "错误", "无效的波特率！")
            self.serial_status_label.setText("串口: 错误")
            self.connect_button.setChecked(False)
        except Exception as e:
            QMessageBox.critical(self, "未知错误", f"连接串口时发生未知错误:\n{e}")
            self.serial_status_label.setText("串口: 错误")
            self.connect_button.setChecked(False)

    def _disconnect_serial(self):
        """断开当前串口连接"""
        # 1. Signal the worker to stop
        if self.serial_worker:
            self.serial_worker.stop()

        # 2. Request the thread to quit (it will finish when the worker loop ends)
        if self.serial_thread and self.serial_thread.isRunning():
            self.serial_thread.quit()
            # Optionally wait for thread to finish, but deleteLater should handle cleanup
            # self.serial_thread.wait(500)

        # 3. Close the serial port instance
        port_name = ""
        if self.serial_instance:
            if self.serial_instance.is_open:
                port_name = self.serial_instance.port
                try:
                    self.serial_instance.close()
                    if port_name: # Log only if port name was valid
                        self._log_serial_info(f"串口 {port_name} 已关闭。\n")
                except serial.SerialException as e:
                    if port_name:
                        self._log_serial_info(f"关闭串口 {port_name} 时出错: {e}\n")
            self.serial_instance = None # Clear the instance variable

        # 4. Update UI
        self.serial_status_label.setText("串口: 已断开")
        self.connect_button.setText("打开串口")
        # Ensure button state is unchecked, even if disconnect was triggered programmatically
        if self.connect_button.isChecked():
            self.connect_button.setChecked(False)

        # Enable settings controls
        self.com_port_combo.setEnabled(True)
        self.baud_rate_combo.setEnabled(True)
        self.refresh_com_btn.setEnabled(True)
        self.send_button.setEnabled(False) # Disable send button

        # 5. Clear thread/worker references (deleteLater handles actual deletion)
        self.serial_thread = None
        self.serial_worker = None


    @pyqtSlot()
    def send_serial_data(self):
        """发送数据到串口"""
        if self.serial_instance and self.serial_instance.is_open:
            data_str = self.send_entry.text()
            if not data_str:
                return # 不发送空内容

            try:
                if self.hex_send_check.isChecked():
                    # 十六进制发送
                    data_str = data_str.replace(" ", "") # 移除空格
                    if len(data_str) % 2 != 0:
                        QMessageBox.warning(self, "警告", "十六进制字符串长度必须为偶数。")
                        return
                    if not all(c in '0123456789abcdefABCDEF' for c in data_str):
                        QMessageBox.warning(self, "警告", "包含无效的十六进制字符。")
                        return
                    try:
                        data_bytes = bytes.fromhex(data_str)
                    except ValueError:
                        QMessageBox.critical(self, "错误", "无效的十六进制字符串。")
                        return
                else:
                    # 文本发送
                    data_bytes = data_str.encode('utf-8') # 使用 UTF-8 编码
                    if self.send_newline_check.isChecked():
                        data_bytes += b'\r\n' # 添加回车换行

                self.serial_instance.write(data_bytes)
                # self._log_serial_info(f"Sent: {data_bytes!r}\n") # 可选：记录发送内容
                # self.send_entry.clear() # 可选：发送后清空输入框

            except serial.SerialTimeoutException:
                QMessageBox.warning(self, "超时", "发送数据超时！")
                self._log_serial_info("发送超时。\n")
            except serial.SerialException as e:
                QMessageBox.critical(self, "串口错误", f"发送数据时出错:\n{e}")
                self._log_serial_info(f"发送错误: {e}\n")
                # 发送错误通常意味着连接有问题，尝试断开
                if self.connect_button.isChecked():
                    self.connect_button.setChecked(False) # 会触发 _disconnect_serial
            except Exception as e:
                QMessageBox.critical(self, "未知错误", f"发送数据时发生未知错误:\n{e}")
                self._log_serial_info(f"发送错误: {e}\n")
        else:
            QMessageBox.warning(self, "警告", "串口未连接！")


    @pyqtSlot(bytes)
    def handle_serial_data(self, data_bytes):
        """处理从串口接收到的数据"""
        cursor = self.receive_text.textCursor()
        is_at_end = cursor.atEnd() # Check if cursor is already at the end

        cursor.movePosition(QTextCursor.MoveOperation.End) # 移动到末尾
        self.receive_text.setTextCursor(cursor)

        if self.hex_display_check.isChecked():
            # 十六进制显示
            hex_string = ' '.join(f'{b:02X}' for b in data_bytes)
            self.receive_text.insertPlainText(hex_string + ' ')
        else:
            # 尝试用 UTF-8 解码显示文本，无法解码的字节用替换符显示
            try:
                # 尝试解码，替换无效字符
                text = data_bytes.decode('utf-8', errors='replace')
                self.receive_text.insertPlainText('[' + get_time_stamp() + '] ' + text)
            except Exception as e:
                # 一般不会到这里，因为 errors='replace'
                self.receive_text.insertPlainText(f"[解码错误: {e}]")

        # Auto-scroll only if the cursor was at the end before insertion
        if is_at_end:
            self.receive_text.moveCursor(QTextCursor.MoveOperation.End)
            # Or using scrollbar:
            # scrollbar = self.receive_text.verticalScrollBar()
            # scrollbar.setValue(scrollbar.maximum())


    @pyqtSlot(str)
    def handle_serial_error(self, error_message):
        """处理串口 Worker 报告的错误"""
        # Avoid showing error if triggered by manual disconnect
        if self.connect_button.isChecked(): # Check if we are supposed to be connected
            QMessageBox.critical(self, "串口错误", error_message)
            self._log_serial_info(f"[错误] {error_message}\n")
            # 发生错误时，强制断开连接
            self.connect_button.setChecked(False) # 会触发 _disconnect_serial

    @pyqtSlot()
    def serial_worker_finished(self):
        """串口 Worker 完成时的槽函数 (用于调试或确认)"""
        # This slot is called when the worker's run() method finishes,
        # either normally (after stop() is called) or due to an error.
        # The disconnect logic should have already handled UI updates and port closing.
        # print("Serial worker finished signal received.")
        pass


    @pyqtSlot()
    def clear_receive_text(self):
        """清空接收文本框"""
        self.receive_text.clear()

    def _log_serial_info(self, message):
        """将普通信息记录到串口接收框"""
        cursor = self.receive_text.textCursor()
        is_at_end = cursor.atEnd()
        cursor.movePosition(QTextCursor.MoveOperation.End)
        self.receive_text.setTextCursor(cursor)
        self.receive_text.insertPlainText(f"[INFO] {message}")
        # Auto-scroll only if the cursor was at the end before insertion
        if is_at_end:
            self.receive_text.moveCursor(QTextCursor.MoveOperation.End)


    # --- Utility Methods ---
    def check_dfu_util_exists(self):
        """检查 dfu-util 是否可用"""
        try:
            process = subprocess.run([DFU_UTIL_COMMAND, "-V"], capture_output=True, text=True, check=False, encoding='utf-8', errors='ignore', creationflags=subprocess.CREATE_NO_WINDOW if sys.platform == 'win32' else 0)
            if process.returncode != 0 and ("not found" in process.stderr.lower() or "不是内部或外部命令" in process.stderr): # 兼容 Windows 提示
                raise FileNotFoundError
            # Log the found version to console, not UI
            if process.stdout:
                print(f"找到 dfu-util: {process.stdout.splitlines()[0]}")
            return True # Indicate success
        except FileNotFoundError:
            QMessageBox.critical(self, "错误", f"未找到 '{DFU_UTIL_COMMAND}' 命令。\n请确保已安装 dfu-util 并将其添加到系统 PATH 环境变量中。")
            self.dfu_status_label.setText("DFU: 错误 (未找到 dfu-util)")
            self._append_dfu_text(f"错误: 未找到 '{DFU_UTIL_COMMAND}'。\n")
            return False # Indicate failure
        except Exception as e:
            QMessageBox.critical(self, "错误", f"检查 dfu-util 时出错: {e}")
            self.dfu_status_label.setText(f"DFU: 错误 ({e})")
            self._append_dfu_text(f"检查 dfu-util 时出错: {e}\n")
            return False # Indicate failure

    def closeEvent(self, event):
        """处理窗口关闭事件"""
        # 停止 DFU 任务（如果正在进行）
        if self.dfu_worker:
            self.dfu_worker.stop() # Signal worker to stop
        if self.dfu_thread and self.dfu_thread.isRunning():
            self.dfu_thread.quit() # Ask thread to quit
            if not self.dfu_thread.wait(500): # Wait max 500ms
                print("Warning: DFU thread did not finish gracefully.")
                # Optionally terminate if needed, but can be risky
                # self.dfu_thread.terminate()

        # 停止串口任务并关闭串口 (use disconnect logic)
        if self.connect_button.isChecked():
            self._disconnect_serial() # Attempt graceful disconnect
        # Ensure thread is stopped even if disconnect failed or wasn't called
        elif self.serial_thread and self.serial_thread.isRunning():
            self.serial_thread.quit()
            if not self.serial_thread.wait(500):
                print("Warning: Serial thread did not finish gracefully.")

        event.accept() # 接受关闭事件


# --- 主程序入口 ---
if __name__ == "__main__":
    # 确保在高 DPI 显示器上表现正常 (可选)
    if hasattr(Qt.ApplicationAttribute, 'AA_EnableHighDpiScaling'):
        QApplication.setAttribute(Qt.ApplicationAttribute.AA_EnableHighDpiScaling, True)
    if hasattr(Qt.ApplicationAttribute, 'AA_UseHighDpiPixmaps'):
        QApplication.setAttribute(Qt.ApplicationAttribute.AA_UseHighDpiPixmaps, True)

    app = QApplication(sys.argv)
    main_window = STM32ToolAppPyQt()
    main_window.show()
    sys.exit(app.exec())

```

