# 异构安全控制系统：PC–ESP32–FPGA–STM32

第十届全国大学生集成电路创新创业大赛 Robei 企业命题作品，获 **华中分赛区二等奖**。

项目以四轮小车为执行载体，将无线通信、安全处理和物理控制分布到 PC、ESP32、FPGA 与 STM32 四个节点，完成从控制指令下发、加密传输、硬件校验到小车执行和状态回传的完整闭环。

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

## 节点职责

### PC 控制端
- 生成控制请求并完成请求侧加密与帧封装；
- 接收返回数据并完成解密、CRC 与 sequence 校验；
- 在完整校验通过后更新设备状态。

### ESP32 通信节点
- 建立 Wi-Fi 连接并完成 UDP 双向通信；
- 在网络数据报与 FPGA UART 数据流之间进行透明桥接。

### FPGA 安全处理节点
- 实现 UART 收发、协议解析和响应封装；
- 完成 CRC、sequence、opcode 等事务检查；
- 集成 AES / SM4 密码模块；
- 在 PC 通信侧与 STM32 执行侧之间维护控制事务。

### STM32 控制节点
- 解析 FPGA 下发的控制帧；
- 通过 PWM / GPIO 驱动电机；
- 生成真实执行状态并返回 FPGA；
- 在通信异常或超时条件下执行安全停车。

## 我的工作

- 担任团队队长，负责系统方案拆分、接口定义与联调推进；
- 负责 FPGA 侧 RTL 开发与系统集成；
- 完成 UART 数据通路、协议处理、事务控制、CRC / sequence 校验及 AES / SM4 接口集成；
- 参与 ESP32 Wi-Fi/UDP–UART 桥接和 STM32 控制链路调试；
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
ESP32 负责网络接入，FPGA 负责安全处理和事务检查，STM32 负责最终物理执行，使通信、安全处理与执行控制保持明确边界。

### 状态闭环
PC 不直接把“命令已发送”视为“动作已完成”。设备状态由 STM32 根据真实执行结果生成，经 FPGA 和 ESP32 返回 PC，再完成完整校验。

### 失联停车
STM32 侧实现独立 fail-safe 逻辑。上游链路异常或超时时，执行端主动停止电机。

## 验证结果

项目采用软件、RTL、实现和实机四层验证：

- Python 回归：**907 项通过，1 项跳过**；
- FPGA pytest：**73 项通过**；
- RTL / 综合后 UART 端到端测试：**10/10 通过**；
- 实现器件：`xc7a50tfgg484-1`；
- 时序结果：**WNS 0.536 ns，WHS 0.006 ns**；
- 完成完整链路失联停车与异常输入验证。

## 文档

- [硬件平台说明](docs/hardware.md)
- [关键设计决策](docs/design-decisions.md)
- [验证方法](docs/verification.md)
- [验证结果摘要](EVIDENCE.md)
- [证据索引](docs/evidence-index.md)
- [架构图源文件](architecture.mmd)
