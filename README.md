# 基于 STM32 与 Modbus RTU 的矿井液压支架监控系统设计

![License](https://img.shields.io/badge/License-GPLv3-blue)
![Platform](https://img.shields.io/badge/Platform-Windows-555)
![Framework](https://img.shields.io/badge/Framework-WPF-purple)
![Language](https://img.shields.io/badge/Language-VB.NET-blueviolet)
![MCU](https://img.shields.io/badge/MCU-STM32F103VET6-2ea44f)
![Protocol](https://img.shields.io/badge/Protocol-Modbus%20RTU-orange)

本项目是一套面向课程设计和工程演示的矿井液压支架模拟监控系统。系统以 STM32F103VET6 为下位机控制核心，以 WPF + VB.NET 上位机作为监控与控制界面，通过 Modbus RTU 串口通信完成状态采集、动作控制、报警判断和历史数据记录。

完整课程设计报告位于：

- [设计报告-基于STM32与Modbus RTU的矿井液压支架监控系统设计.pdf](<./设计报告-基于STM32与Modbus RTU的矿井液压支架监控系统设计.pdf>)

报告围绕煤矿综采工作面液压支架动作控制和状态监测需求，完成了系统总体方案、硬件电路、PCB/BOM、STM32 下位机软件、WPF 上位机软件、Modbus RTU 通信协议、系统调试测试和总结展望等内容。当前仓库保存了报告中对应的完整工程文件和展示材料。

## 克隆与部署

本仓库计划作为总项目仓库 `hydraulic-support-system` 使用，`hardware/` 与 `software/` 通过 Git submodule 引用两个独立子仓库：

- `stm32_hydraulic-support-controller`：STM32F103VET6 下位机控制器。
- `wpf-vb_hydraulic-support-monitoring`：WPF + VB.NET 上位机监控软件。

首次克隆推荐使用：

```bash
git clone --recurse-submodules https://github.com/Rowen-Nell/hydraulic-support-system.git
```

如果已经普通克隆，则进入仓库后执行：

```bash
git submodule update --init --recursive
```

开发时按技术栈分别打开子目录：

```text
hardware/  -> 使用 VS Code + STM32-for-VSCode / CMake / CubeMX 工作流
software/  -> 使用 Visual Studio 或 VS Code 打开 WPF VB.NET 项目
```

更新子模块到远端最新提交：

```bash
git submodule update --remote --merge
```

当子模块更新后，外层总仓库还需要提交新的 submodule 指针，确保其他人克隆时拿到同一版本的上下位机工程。
## 系统展示

### 实物与驱动器

![系统整体实物](<.images/材料 - 系统整体.jpg>)

![TB6600 驱动器与步进电机](<.images/材料 - TB6600.jpg>)

### 总体架构与软件分层

![系统总体结构框图](<.images/材料 - 01-系统总体结构框图.png>)

![下位机软件分层结构图](<.images/材料 - 02-下位机软件分层结构图.png>)

### 通信与控制流程

![Modbus RTU 通信流程图](<.images/材料 - 03-Modbus_RTU通信流程图.png>)

![升降电机控制流程图](<.images/材料 - 04-升降电机控制流程图.png>)

![前进后退目标位置控制流程图](<.images/材料 - 05-前进后退目标位置控制流程图.png>)

![压力模型流程图](<.images/材料 - 06-压力模型流程图.png>)

### 上位机典型显示状态

![上位机角度 -13 度显示](<.images/材料 - 上位机角度-13.png>)

![上位机角度 0 度显示](<.images/材料 - 上位机角度0.png>)

![上位机角度 8 度警告状态](<.images/材料 - 上位机角度8 警告.png>)

![上位机角度 14 度紧急状态](<.images/材料 - 上位机角度14 紧急.png>)

## 目录结构

```text
system_motor-control/
├── README.md
├── 设计报告-基于STM32与Modbus RTU的矿井液压支架监控系统设计.pdf
├── .images/               # README 与报告展示图片
├── software/              # WPF + VB.NET 上位机软件
├── hardware/              # STM32F103VET6 下位机固件、PCB资料、开发文档
```

说明：

- `software/` 和 `hardware/` 以 Git submodule 方式管理，分别保留各自独立 Git 历史。`software/` 是 WPF/VB.NET 上位机项目，`hardware/` 是 STM32 下位机项目。
- 更详细的子项目说明见 [software/README.md](./software/README.md) 和 [hardware/README.md](./hardware/README.md)。

## 设计目标

本系统不直接接入真实液压执行机构和真实压力传感器，而是使用 STM32、TB6600、步进电机和虚拟传感器模型搭建一个可调试、可观察、可演示的教学型监控平台。

主要目标包括：

- STM32 作为 Modbus RTU 从站，WPF 上位机作为 Modbus RTU 主站。
- 上位机通过串口读取角度、压力 1、压力 2，并显示实时曲线和报警状态。
- 上位机下发升高、降低、前进、后退命令。
- STM32 控制两个 TB6600 步进驱动器，分别模拟移动轴和升降轴。
- STM32 内部生成虚拟角度与压力模型，不依赖真实传感器。
- 通过 SQLite 保存上位机历史采样数据。
- 形成从硬件、固件、通信到上位机界面的完整闭环。

## 系统组成

系统由四层组成：

| 层级 | 组成 | 职责 |
| --- | --- | --- |
| 上位机监控层 | WPF + VB.NET 软件 | 串口配置、设备轮询、数据显示、动作控制、报警日志、历史数据 |
| 现场总线通信层 | Modbus RTU / USART1 | 主从请求响应、CRC 校验、寄存器和线圈映射 |
| 下位机控制层 | STM32F103VET6 固件 | 协议解析、动作映射、步进电机控制、虚拟传感器模型 |
| 执行与模拟层 | TB6600、步进电机、虚拟压力/角度 | 模拟液压支架前进、后退、升高、降低和状态变化 |

核心硬件：

- STM32F103VET6 开发板
- TB6600 步进电机驱动器 x 2
- 42 步进电机 x 2
- 外部 12V 电源
- PC 上位机

## 上位机软件

目录：[software](./software)

上位机采用 WPF + VB.NET，整体遵循 MVVM 思路：

- `Models/`：`SupportModel`、`AlarmModel` 等数据模型。
- `ViewModels/MainViewModel.vb`：界面绑定、定时轮询、命令执行和报警判断核心。
- `Services/ModbusService.vb`：NModbus4 主站通信封装。
- `Services/DatabaseService.vb`：SQLite 历史数据存储。
- `Views/MainWindow.xaml`：主监控界面。
- `Views/SerialConfigWindow.xaml`：串口配置窗口。

主要功能：

- 支持串口选择和 `115200 8N1` 参数连接。
- 支持 56 个支架节点界面模型。
- 周期读取从站输入寄存器 `0..2`。
- 将角度寄存器还原为 `registers(0) - 15.0`。
- 将压力寄存器还原为 `registers(1) / 100.0` 和 `registers(2) / 100.0`。
- 发送升高、降低、前进、后退动作命令。
- 通过 `SyncLock` 管理串口总线，避免轮询线程和人工控制命令冲突。
- 根据压力 1 判断正常、警告和紧急状态。
- 将历史数据写入本地 SQLite 数据库。

报警规则：

| pressure1 | 状态 |
| --- | --- |
| `< 30.0 MPa` | 正常 |
| `>= 30.0 MPa` 且 `< 35.0 MPa` | 警告 |
| `>= 35.0 MPa` | 紧急 |

## 下位机固件

目录：[hardware](./hardware)

下位机基于 STM32F103VET6，使用 CubeMX、CMake 和 STM32-for-VSCode 工作流。固件采用三层结构：

| 层级 | 目录 | 说明 |
| --- | --- | --- |
| Core | `hardware/Core/` | CubeMX 生成代码，保留基础初始化和 `App_Init()` / `App_Loop()` 调用 |
| Lib | `hardware/Lib/` | 通用底层库，包括步进电机、按键、Modbus RTU 协议 |
| App | `hardware/App/` | 液压支架业务，包括动作控制、传感器模拟、Modbus 应用映射 |

关键模块：

- `App/Src/app.c`：应用入口，选择按键测试模式或 Modbus 正式模式。
- `App/Src/support_motion.c`：将升高、降低、前进、后退转换为电机动作。
- `App/Src/sensor_simulator.c`：维护角度、压力 1、压力 2。
- `App/Src/modbus_app.c`：映射 Modbus 寄存器和线圈到业务动作。
- `Lib/Src/stepper_driver.c`：TIM2 PWM 脉冲输出、方向控制、busy 保护。
- `Lib/Src/modbus_rtu.c`：RTU 帧接收、CRC16、功能码解析和响应。

## 硬件连接

已验证的 TB6600 接线方式：

```text
移动轴：
PA0  -> PUL-
PD12 -> DIR-
PUL+ / DIR+ -> 3.3V

升降轴：
PA1  -> PUL-
PD14 -> DIR-
PUL+ / DIR+ -> 3.3V

TB6600 电机电源：
外部 12V 电源供电，不由 STM32 开发板给电机供电。
```

主要 STM32 引脚：

| 功能 | 引脚 | 说明 |
| --- | --- | --- |
| 移动电机 STEP | PA0 | TIM2 CH1 PWM 输出 |
| 升降电机 STEP | PA1 | TIM2 CH2 PWM 输出 |
| 移动电机 DIR | PD12 | 方向控制 |
| 移动电机 ENA | PD13 | 使能预留 |
| 升降电机 DIR | PD14 | 方向控制 |
| 升降电机 ENA | PD15 | 使能预留 |
| USART1 TX/RX | PA9 / PA10 | Modbus RTU 通信 |
| 按键 | PE13 / PE14 / PE15 | 升高、降低、移动测试 |
| LED | PB6 / PB7 / PB8 | 运行、电机、通信指示 |

`ENA+` / `ENA-` 当前可以不接，代码中仍保留 ENA 极性配置。若后续要释放停机保持电流，可补接 ENA 并重新验证驱动器极性。

## Modbus RTU 协议

串口参数：

| 项目 | 值 |
| --- | --- |
| 从站地址 | `1` |
| 波特率 | `115200` |
| 数据位 | `8` |
| 校验 | 无 |
| 停止位 | `1` |
| 协议 | Modbus RTU |

支持功能码：

| 功能码 | 用途 |
| --- | --- |
| `04` | 读输入寄存器 |
| `05` | 写单个线圈 |
| `06` | 写单个保持寄存器 |

地址映射：

| 地址类型 | 地址 | 含义 | 数据格式 |
| --- | --- | --- | --- |
| 输入寄存器 | `0` | 支架显示角度 | `actual_angle + 15` |
| 输入寄存器 | `1` | 压力 1 | MPa x 100 |
| 输入寄存器 | `2` | 压力 2 | MPa x 100 |
| 保持寄存器 | `1` | 下降参数 | 兼容保留 |
| 保持寄存器 | `2` | 上升参数 | 兼容保留 |
| 保持寄存器 | `3` | 前进 / 后退目标位置 | `uint16` |
| 线圈 | `0` | 下降触发 | ON 触发 |
| 线圈 | `1` | 上升触发 | ON 触发 |
| 线圈 | `2` | 移动触发 | ON 触发 |

角度寄存器采用偏移编码，因为 Modbus 输入寄存器是 16 位无符号值：

```text
实际角度 -15° -> 输入寄存器 0 = 0
实际角度   0° -> 输入寄存器 0 = 15
实际角度 +15° -> 输入寄存器 0 = 30
```

## 运动与传感器模型

### 前进 / 后退

上位机并不直接发送前进或后退方向码，而是写保持寄存器 `3` 作为目标位置，再写线圈 `2` 触发移动。STM32 根据目标位置和当前位置比较决定方向：

```text
targetPosition > currentPosition -> 前进
targetPosition < currentPosition -> 后退
targetPosition == currentPosition -> 不动作
```

移动轴映射关系：

```text
位置差 10 -> 80 脉冲
STEP 频率 -> 1000 Hz
```

只有 `Stepper_MoveSteps()` 返回 `HAL_OK` 后，STM32 才更新内部 `currentPosition`，避免电机未启动时上下位机状态不同步。

### 升高 / 降低

升降轴使用显示角度到累计脉冲的映射：

```text
显示角度范围：-15°..+15°
启动默认角度：0°
每次升高：显示角度 +1°
每次降低：显示角度 -1°
机械角度范围：-720°..+720°
TB6600 8 细分：1600 脉冲 = 360°
升降累计脉冲范围：-3200..+3200
```

达到 `+15°` 后继续升高不会再启动电机；达到 `-15°` 后继续降低不会再启动电机。

### 虚拟压力

压力由当前显示角度统一计算，前进 / 后退不独立改变压力：

| 角度范围 | pressure1 |
| --- | --- |
| `angle <= -13°` | `0.00 MPa` |
| `-13° < angle <= 0°` | 从 `0.00 MPa` 线性增加到 `26.00 MPa` |
| `0° < angle < +13°` | 从 `26.00 MPa` 线性增加到 `35.00 MPa` |
| `angle = +13°` | `35.00 MPa` |
| `angle = +14°` | 约 `37.00 MPa` |
| `angle = +15°` | `40.00 MPa` |

```text
pressure2 = pressure1 x 0.95
```

## 构建与运行

### 上位机

推荐使用 Visual Studio 打开：

```text
software/WPF_VB_Monitoring.slnx
```

运行要求：

- Windows 10/11
- .NET 10.0 桌面运行时
- 可用串口或虚拟串口

仿真测试可使用虚拟串口工具建立串口对，并使用 Modbus Slave 软件模拟下位机寄存器。

### 下位机

推荐使用 STM32-for-VSCode 打开：

```text
hardware/
```

命令行构建：

```powershell
cd hardware
cmake --build build/Debug
```

普通 PowerShell 中找不到 ARM GCC 可能是正常现象；实际构建可依赖 STM32-for-VSCode 扩展环境。

## 调试与测试结果

系统调试按以下顺序完成：

1. 单模块测试：检查 STM32 外设、GPIO、TIM2 PWM、USART1 和基础工程。
2. 下位机独立测试：启用按键测试模式，验证两个 TB6600、两个步进电机、方向和 busy 保护。
3. 通信联调：启用 Modbus 模式，使用 WPF 上位机读取输入寄存器并触发线圈动作。
4. 整体稳定性测试：连续动作、边界动作、压力模型、断开重连和快速误操作测试。

报告中的系统功能测试结果均为通过：

| 测试项目 | 预期结果 | 实际结果 |
| --- | --- | --- |
| 串口连接 | 上位机连接成功 | 通过 |
| 输入寄存器读取 | 显示角度、压力 1、压力 2 | 通过 |
| 升高控制 | 角度 +1°，升降电机正向转动 | 通过 |
| 降低控制 | 角度 -1°，升降电机反向转动 | 通过 |
| 角度上限 | 达到 +15° 后电机不再动作 | 通过 |
| 角度下限 | 达到 -15° 后电机不再动作 | 通过 |
| 压力模型 | 角度 0°、+13°、+15° 对应约 26、35、40 MPa | 通过 |
| 前进后退 | 转角随 MoveStep 变化，方向正确 | 通过 |
| 断开重连 | 重新打开串口后恢复通信 | 通过 |

当前下位机构建记录约为：

```text
FLASH: 19152 B / 512 KB
RAM:   2072 B / 64 KB
```

## 当前状态与已知问题

当前系统已完成：

- PCB 原理图、PCB Layout、BOM 和实物装配资料整理。
- STM32F103VET6 CubeMX / CMake 工程配置。
- TB6600 + 步进电机按键测试。
- Modbus RTU 从站协议和应用映射。
- WPF 上位机 Modbus 主站、实时显示、报警和历史数据存储。
- 前进 / 后退目标位置差值逻辑修正。
- 升降角度边界和累计脉冲映射修正。
- 虚拟压力模型修正。
- 上下位机联调和稳定性测试。

已知保留问题：

- 移动轴 TB6600 停机后曾出现发热和滋滋声。由于当前 ENA 未接，固件无法真正释放驱动器保持电流；该问题更可能来自驱动器个体、保持电流策略或硬件状态，不作为当前软件阻塞项。

## 后续扩展方向

报告中给出的后续改进方向包括：

- 将虚拟角度和压力替换为真实压力变送器、倾角传感器或位移传感器。
- 将串口 Modbus RTU 扩展为真实 RS485 多节点总线。
- 为多个 STM32 节点分配不同从站地址，验证 56 台支架轮询性能。
- 增加紧急停止线圈、故障码寄存器、动作完成反馈和通信异常统计。
- 将上位机进一步扩展为更接近工业 SCADA 的界面，增加权限管理、数据导出、报警确认和历史报表。

## 资料索引

- [hardware/README.md](./hardware/README.md)：下位机工程详细说明。
- [software/README.md](./software/README.md)：上位机软件详细说明。
- `hardware/Doc/PCB 原理图.pdf`：PCB 原理图。
- `hardware/Doc/PCB Layout.pdf`：PCB 布局图。
- `hardware/Doc/PCB BOM.xlsx`：PCB 物料清单。
- `hardware/Doc/STM32F103VET6 数据手册.pdf`：芯片数据手册。
- `hardware/Doc/初始开发指导文档.md`：下位机开发指导文档。
- `.images/`：系统展示图、控制流程图和上位机状态截图。

