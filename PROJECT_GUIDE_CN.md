# Dummy-Robot 本地工程指南

> 稚晖君（彭志辉）开源六轴迷你机械臂项目  
> 官方仓库：https://github.com/peng-zhihui/Dummy-Robot  
> Gitee 镜像：https://gitee.com/peng_zhihui/Dummy-Robot  
> B 站视频：[【自制】我造了一台钢铁侠的机械臂！【硬核】](https://www.bilibili.com/video/BV12341117rG)

本目录由 ZIP 下载于 2026-06-28 初始化，路径：`d:\Code\Dummy-Robot`。

---

## 项目简介

Dummy-Robot 是一套 **超迷你工业风格六轴机械臂** 完整开源方案，包含：

| 模块 | 说明 |
|------|------|
| 机械结构 | 步进电机 + 谐波减速（原版）；另有摆线减速青春版规划 |
| 通信 | CAN 总线串联各关节驱动（电源 2 线 + CAN 2 线） |
| 核心控制器 REF | STM32F4 + FreeRTOS，运动学/轨迹规划 |
| 步进驱动 Ctrl-Step | STM32F1 闭环步进，20/42 两种规格 |
| 上位机 | DummyStudio（Unity）、CLI-Tool、RoboDK 驱动 |
| 示教器 | Peak（Git Submodule，需单独拉取） |

---

## 目录结构

```
Dummy-Robot/
├── 0.SubModules/          # 子模块（Peak 无线示教器）
├── 1.Hardware/            # PCB 与硬件工程（REF、驱动器、夹爪等）
├── 2.Firmware/            # 固件源码与预编译 HEX
│   ├── Core-STM32F4-fw/   # REF 核心控制器固件（CLion/STM32CubeIDE）
│   ├── Ctrl-Step-Driver-STM32F1-fw/  # 步进电机驱动固件
│   ├── _Released HEX/     # 可直接烧录的 hex
│   └── CLI-Tool/          # reftool 命令行调试工具
├── 3.Software/            # DummyStudio 上位机等
├── 4.Model/               # 3D 模型、机箱、手提箱等
├── 5.Docs/                # 文档与图片
└── README.md              # 官方说明（英文为主）
```

---

## 固件工程入口（二次开发）

| 工程 | 路径 | MCU | 工具链 |
|------|------|-----|--------|
| **核心控制器** | `2.Firmware/Core-STM32F4-fw/` | STM32F4 | CMake + CLion / STM32CubeIDE |
| **步进驱动 42** | `2.Firmware/Ctrl-Step-Driver-STM32F1-fw/` | STM32F103 | CMake + CLion |
| **预编译固件** | `2.Firmware/_Released HEX/` | — | ST-Link 直接烧录 |

核心固件分层：

- `Robot/` — 运动学、DummyRobot 类、DH 参数
- `UserApp/` — 应用层与协议（CAN/USB/串口）
- `BSP/` — OLED、IMU、蜂鸣器等板载驱动

---

## 硬件子工程（1.Hardware）

- `REF` — 核心板 + 底板
- `MotorDriver-20` / `MotorDriver-42` — 步进驱动 PCB
- `HandModule` — 夹爪
- `LedRing` — LED 灯环
- `Dangle` — 无线空间定位控制器
- `Controller` — 相关控制器硬件

设计工具：Altium Designer（见官方 README）。

---

## 快速上手路径

### 仅体验 / 烧录固件

1. 硬件按官方接线（CAN 串联、电源分配）。
2. 电机驱动：烧录 `Ctrl-Step-Driver-STM32F1-fw.hex`，首次上电编码器校准。
3. 核心板：烧录 `Core-STM32F4-fw` 对应 hex。
4. 用 `CLI-Tool` 或 DummyStudio 发指令测试。

### 修改核心逻辑

1. 用 **CLion** 或 **STM32CubeIDE** 打开 `Core-STM32F4-fw`。
2. 阅读 `Robot/instances/dummy_robot.h` 配置 DH 参数与关节驱动信息。
3. 二次开发重点目录：`UserApp/`。

### 修改电机驱动

1. 打开 `Ctrl-Step-Driver-STM32F1-fw`。
2. 协议见 `UserApp/protocols/interface_can.cpp`、`interface_uart.cpp`。

---

## Submodule：Peak 示教器

官方通过 submodule 引用 Peak 仓库：

```text
https://github.com/peng-zhihui/Peak
```

ZIP 下载不包含 submodule。若需 Peak 源码，请：

```powershell
# 需安装 Git 后执行
cd d:\Code\Dummy-Robot
git clone https://github.com/peng-zhihui/Dummy-Robot.git
git submodule update --init --recursive
```

或单独下载 Peak 到 `0.SubModules/Peak`。

---

## 相关开源项目

| 项目 | 链接 | 关系 |
|------|------|------|
| CycloidAcuratorNano | https://github.com/peng-zhihui/CycloidAcuratorNano | 低成本摆线减速器（青春版） |
| XDrive | https://github.com/unlir/XDrive | 步进闭环驱动参考 |
| Peak | https://github.com/peng-zhihui/Peak | 无线示教器 |

---

## 成本与复现提示（来自官方）

- **原版**：谐波减速器成本高（二手约 600 元/个），加工难度大。
- **青春版（规划中）**：3D 打印 + 摆线减速，目标整机 2000 元以内，软件与原版通用。

新手建议关注稚晖君后续 **Dummy 青春版** 与教程视频。

---

## 本机环境建议

- **烧录**：ST-Link + STM32CubeProgrammer
- **IDE**：CLion（CMake 工程）或 STM32CubeIDE
- **上位机**：Windows 运行 DummyStudio（`3.Software`）
- **Git**：建议安装 Git 以便同步更新与子模块

---

## 更新仓库

```powershell
# 安装 Git 后
cd d:\Code
git clone --recursive https://github.com/peng-zhihui/Dummy-Robot.git
# 或
cd d:\Code\Dummy-Robot
git pull
git submodule update --init --recursive
```

当前目录为 ZIP 快照，无 `.git` 历史。
