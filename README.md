# 异构安全控制系统：PC–ESP32–FPGA–STM32

这是第十届全国大学生集成电路创新创业大赛 Robei 企业命题项目的展示仓库，项目获 **华中分赛区二等奖**。

系统以四轮小车为执行载体，将无线通信、安全处理与物理控制拆分到 PC、ESP32、FPGA 和 STM32 四个节点，形成完整的双向控制链路。

## 系统架构

![系统架构](architecture.svg)

```text
PC
  │ Wi-Fi / UDP
  v
ESP32
  │ UART
  v
FPGA
  │ UART
  v
STM32
  │
  v
电机 / 小车

状态回传：
STM32 → FPGA → ESP32 → PC
```

### PC

- 生成控制请求；
- 完成请求侧加密与帧封装；
- 接收返回数据并进行解密、CRC 与 sequence 校验；
- 只在完整校验通过后更新设备状态。

### ESP32

- 负责 Wi-Fi / UDP 通信；
- 在网络数据报与 UART 数据流之间进行双向桥接；
- 不承担小车控制状态机。

### FPGA

- 实现 UART 数据通路；
- 完成协议解析与响应封装；
- 执行 CRC、sequence、opcode 等检查；
- 集成 AES / SM4 密码模块；
- 维护控制事务状态，并连接 PC 侧通信与 STM32 执行侧。

### STM32

- 解析 FPGA 下发的控制帧；
- 通过 PWM / GPIO 驱动电机；
- 生成真实执行状态；
- 在通信异常或超时条件下执行安全停车。

## 我的主要工作

- 担任团队队长，负责系统方案拆分、接口定义和联调推进；
- 负责 FPGA 侧 RTL 开发与系统集成；
- 完成 UART、协议处理、事务控制、CRC / sequence 校验及 AES / SM4 接口集成；
- 参与 ESP32 与 STM32 端接口调试；
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

通信、安全处理和物理执行分别位于不同节点：

- ESP32 负责网络接入；
- FPGA 负责安全事务和协议检查；
- STM32 负责最终物理执行。

这样可以避免把通信、状态判断和电机控制集中在单一节点。

### 真实状态回传

PC 不直接把“命令已发送”当作“动作已完成”。

系统状态由 STM32 根据真实执行结果生成，经 FPGA 和 ESP32 返回 PC，再完成完整校验后更新。

### 失联停车

STM32 侧保留独立的 fail-safe 逻辑。即使上游 Wi-Fi、ESP32 或 FPGA 链路异常，执行端也能在超时后主动停止电机。

## 验证

项目采用分层验证：

- **软件层**：协议、密码调用、状态过滤与异常路径；
- **RTL 层**：FPGA 模块与端到端 UART 数据通路；
- **实现层**：综合、布局布线、时序与 DRC；
- **实机层**：完整控制链路、异常输入与失联停车。

代表性验证结果：

- Python 回归：**907 项通过，1 项跳过**；
- FPGA pytest：**73 项通过**；
- RTL / 综合后 UART 端到端测试：**10/10 通过**；
- 实现器件：`xc7a50tfgg484-1`；
- 时序：**WNS 0.536 ns，WHS 0.006 ns**。

详细证据见：

- [验证方法](docs/verification.md)
- [验证结果摘要](EVIDENCE.md)
- [证据索引](docs/evidence-index.md)

## 相关代码

FPGA 加密网关的公开 RTL、testbench、构建脚本和板级调试工具见：

**LIZH-DEVE/FPGA-Crypto-Gateway**

## 文档

- [硬件平台](docs/hardware.md)
- [关键设计决策](docs/design-decisions.md)
- [验证方法](docs/verification.md)
- [证据索引](docs/evidence-index.md)
- [架构图源文件](architecture.mmd)
