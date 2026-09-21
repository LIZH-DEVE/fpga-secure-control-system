# 异构安全控制系统：PC–ESP32–FPGA–STM32

第十届全国大学生集成电路创新创业大赛 Robei 企业命题作品，获 **华中分赛区二等奖**。

项目以四轮小车为执行载体，将网络通信、安全处理与物理控制分布到 PC、ESP32、FPGA 和 STM32 四个节点，完成控制指令下发、加密传输、硬件检查、执行控制和状态回传的完整闭环。

## 系统架构

![系统架构](architecture.svg)

```text
PC 控制端
   │ Wi-Fi / UDP
   v
ESP32 通信节点
   │ UART
   v
FPGA 安全处理节点
   │ UART
   v
STM32 控制节点
   │
   v
电机 / 四轮小车

状态回传：
STM32 → FPGA → ESP32 → PC
```

## 节点分工

| 节点 | 主要职责 |
| --- | --- |
| **PC** | 控制请求生成、请求侧加密与封装、返回帧校验、状态显示 |
| **ESP32** | Wi-Fi / UDP 与 UART 之间的双向通信桥接 |
| **FPGA** | UART、协议解析、事务控制、CRC / sequence 检查、AES / SM4 接口集成 |
| **STM32** | 控制帧解析、PWM / GPIO 电机驱动、执行状态回传、失联停车 |

## 负责内容

- 担任团队队长，负责系统方案拆分、接口定义与联调推进；
- 负责 FPGA 侧 Verilog / RTL 开发与系统集成；
- 完成 UART 数据通路、协议处理、事务控制、CRC / sequence 检查及 AES / SM4 接口集成；
- 参与 ESP32 Wi-Fi/UDP–UART 桥接与 STM32 控制链路调试；
- 建立软件、RTL、综合实现和实机链路的分层验证流程。

## 硬件平台

![硬件平台概览](docs/images/hardware-overview.jpg)

| 节点 | 平台 |
| --- | --- |
| FPGA | Robei Artix-7 / Xilinx XC7A50T |
| 无线通信 | ESP32-WROOM |
| 控制 MCU | STM32F103 |
| 执行机构 | 四轮小车底盘 + 电机驱动 |

## 关键设计

### 分层控制

ESP32 负责网络接入，FPGA 负责协议与安全处理，STM32 负责最终物理执行。通信、安全处理与执行控制分别位于清晰的节点边界内，便于分段调试和故障定位。

### 状态闭环

PC 不直接把“命令已发送”作为“动作已完成”。设备状态由 STM32 根据真实执行结果生成，经 FPGA 和 ESP32 返回 PC，并在完成返回帧校验后更新界面状态。

### 失联停车

STM32 独立监测控制链路。上游通信中断或超过超时时间后，执行端主动停止电机并将 PWM 置零。

## 验证

| 层级 | 结果 |
| --- | --- |
| 软件协议与异常路径回归 | 907 passed，1 skipped |
| FPGA 自动化测试 | 73 passed |
| RTL UART 端到端路径 | 10/10 |
| 综合后 UART 端到端路径 | 10/10 |
| Implementation timing | WNS 0.536 ns，WHS 0.006 ns |
| 实机安全路径 | 完整链路失联停车与异常输入验证通过 |

测试层级、版本关系和实机验证条件见 [验证说明](docs/verification.md)。

## 文档

- [硬件平台](docs/hardware.md)
- [设计决策](docs/design-decisions.md)
- [验证说明](docs/verification.md)
- [架构图源文件](architecture.mmd)

## FPGA 源码

RX50T FPGA 加密网关的 RTL、testbench、Vivado 构建脚本和板级调试工具位于：

**[LIZH-DEVE/FPGA-Crypto-Gateway](https://github.com/LIZH-DEVE/FPGA-Crypto-Gateway)**
