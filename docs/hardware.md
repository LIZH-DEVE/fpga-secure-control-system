# 硬件平台

## 系统组成

系统由无线通信、FPGA 事务处理、嵌入式执行和独立供电四部分组成。各节点承担独立职责，并通过 UART / Wi-Fi 形成完整控制链路。

![硬件平台概览](images/hardware-overview.jpg)

上图依次为 FPGA 开发板、ESP32 通信模块、STM32 控制板和电源模块。

## FPGA 平台

- **开发板**：Robei Artix-7 FPGA 开发板
- **器件**：Xilinx Artix-7 XC7A50T，`fgg484-1`
- **主要职责**：
  - UART frame RX / TX
  - AES / SM4 接口集成
  - opcode、CRC、sequence 检查
  - 车控事务管理
  - STM32 状态检查与响应生成

FPGA 位于无线通信节点和物理执行节点之间，将协议处理、安全检查和车控事务集中在同一个硬件边界内。

## ESP32

- **模块**：ESP32-WROOM
- **网络接口**：Wi-Fi / UDP
- **设备接口**：UART
- **主要职责**：
  - 建立 Wi-Fi 连接
  - 接收 PC 端 UDP 数据报并转发至 FPGA
  - 将 FPGA 返回数据封装为 UDP 数据报发送给 PC
  - 维护通信链路，不保存独立车控业务状态

ESP32 作为网络接入和串口桥接节点，使网络通信与后续协议 / 控制逻辑保持分离。

## STM32

- **控制器**：STM32F103
- **主要职责**：
  - 解析 FPGA 下发的控制帧
  - PWM / GPIO 电机控制
  - 生成真实执行状态并回传
  - 监测通信超时
  - 异常状态下执行停车并将 PWM 置零

STM32 直接控制执行器，因此失联停车逻辑部署在该节点。

## 电机与供电

移动平台使用四轮底盘和独立电源模块。

硬件联调重点检查：

- 电机启动和换向时的供电压降；
- FPGA、ESP32 和 STM32 共地；
- 各节点逻辑电平与 UART 接线；
- 电源模块输入 / 输出状态；
- 供电异常是否触发复位或错误停车。

## 接口关系

```text
PC
 │ Wi-Fi / UDP
ESP32
 │ UART
FPGA
 │ UART CONTROL / STATUS
STM32
 │ PWM / GPIO
Motor
```

状态沿反方向返回：

```text
STM32 STATUS
  → FPGA validation / response
  → ESP32 UDP forwarding
  → PC decryption / state update
```

超声波接口在项目中完成了接口预留和软件 / RTL 模拟，实体传感器未安装。
