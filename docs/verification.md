# 验证说明

项目按软件、RTL、综合实现和真实硬件四个层级进行验证。不同层级用于回答不同问题，因此结果分别记录。

## 软件

协议处理、密码调用封装、帧校验、状态过滤和异常路径使用 Python 自动化测试。

记录结果：

```text
907 passed
1 skipped
```

## FPGA / RTL

FPGA 侧接口与事务逻辑使用自动化测试和 RTL testbench 验证。

记录结果：

```text
FPGA automated tests: 73 passed
UART RX → protocol / transaction path → UART TX: 10/10
```

UART 端到端测试从真实 UART 接口进入并观察 UART 输出，不通过强制内部状态绕过主数据通路。

## 综合实现

目标器件：

```text
xc7a50tfgg484-1
```

记录结果：

| 项目 | 结果 |
| --- | ---: |
| Post-synthesis UART end-to-end | 10/10 |
| WNS | 0.536 ns |
| WHS | 0.006 ns |

对应实现检查未记录 black box、NSTD、UCIO、critical warning 或 DRC error。

## 实机

真实硬件测试覆盖：

- AES / SM4 STOP 压力路径；
- STM32 直连停车；
- PC–ESP32–FPGA–STM32 完整链路失联停车；
- 多类异常输入与最终 PWM 归零。

其中完整链路失联停车记录为：

```text
10 / 10
```

AES 和 SM4 的 STOP 压力测试分别记录 600 次通过。

## 版本关系

软件回归、RTL / post-synthesis 验证和真实硬件测试并非全部来自同一个 bitstream 或同一开发阶段。

完整开发基线用于验证 AES / SM4、协议与 RTL 主路径；比赛后期还存在针对现场需求的精简配置。因此这里按测试层级和产物分别描述结果，不把不同版本的数字合并为单一“整机测试”。

## 可追溯信息

完整工程记录中保留：

- Git commit SHA；
- STM32 firmware hash；
- FPGA bitstream hash；
- Vivado build / timing report；
- 软件与 RTL test logs；
- 实机异常路径与安全停车记录。
