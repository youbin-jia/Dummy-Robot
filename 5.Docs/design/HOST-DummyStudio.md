# HOST-DummyStudio 详细设计

## 1. 模块定位

Unity **预编译上位机**：提供示教/调试类图形界面，通过串口与 REF 通信。仓库内**无 C# 源码**，本文仅说明部署、配置与在总体架构中的角色。

**代码路径**：[`3.Software/DummyStudio/`](../../3.Software/DummyStudio/)

## 2. 在总体架构中的位置

对应总体架构上位机 **DummyStudio**：经 UART ASCII（或厂商约定串口路径）连接 REF。

```mermaid
flowchart LR
  Studio[DummyStudio] -->|UART_115200| Proto[REF_Protocols_ASCII]
  Proto --> Robot[REF_Robot]
```

| 方向 | 邻居模块 | 交互内容 |
|------|----------|----------|
| 下游 | [REF-Protocols](REF-Protocols.md) | ASCII 指令（START/STOP、MoveJ/L 等） |
| 并列 | [HOST-CLI-Tool](HOST-CLI-Tool.md) | CLI 偏 Fibre；Studio 偏 GUI |

## 3. 内部结构

```text
DummyStudio/
├── DummyStudio_Data/           # Unity 构建数据
│   ├── StreamingAssets/
│   │   └── serial_config.txt   # 串口 VID / 波特率
│   ├── app.info                # pengzhihui / DummyStudio
│   └── boot.config
├── MonoBleedingEdge/           # Mono 运行时（含 Linux 相关）
└── （可执行文件可能需从官方 Release 补全）
```

本快照可能**缺少 `.exe`**，需从官方发布包补齐后再运行。

## 4. 关键类与入口

无开源类可列。运行入口为平台可执行文件（Windows 下通常为 `DummyStudio.exe`）。

## 5. 核心行为 / 数据流

```text
启动 DummyStudio
  → 读取 StreamingAssets/serial_config.txt
  → 按 VendorID / BaudRate 打开串口
  → GUI 示教 / 轨迹 → ASCII 下发 REF
```

具体 UI 功能以官方视频与 Release 说明为准；无法从本仓库源码级验证。

## 6. 对外接口

配置文件接口：

**路径**：`DummyStudio_Data/StreamingAssets/serial_config.txt`

```text
VendorID=1209
BaudRate=115200
```

与 REF USB/CDC 或串口桥的 VID、波特率需匹配。ASCII 命令语义见 [ARCHITECTURE §4.3](../ARCHITECTURE_CN.md) 与 [REF-Protocols](REF-Protocols.md)。

## 7. 配置与约束

- 源码未开源：无法在本仓库内做 UI/协议层二次编译
- 若仅有 `DummyStudio_Data` 而无 exe，程序无法启动
- `VendorID=1209` 为 ODrive 生态常用测试 VID 之一，需与固件 USB 描述符一致

## 8. 二次开发要点

- **可做**：改 `serial_config.txt`；用 [HOST-CLI-Tool](HOST-CLI-Tool.md) 或自写串口脚本替代 GUI
- **不可做**：在本仓库内修改 Studio 业务逻辑（无源码）
- 需要图形示教且要改协议时：优先扩展 REF ASCII/Fibre，再考虑自研上位机

## 9. 相关文档

- [系统架构](../ARCHITECTURE_CN.md) §6.2
- [设计文档索引](README.md)
- [HOST-CLI-Tool](HOST-CLI-Tool.md) · [REF-Protocols](REF-Protocols.md)
