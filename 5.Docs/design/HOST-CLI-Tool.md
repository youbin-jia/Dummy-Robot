# HOST-CLI-Tool 详细设计

## 1. 模块定位

PC 端 **Python 交互工具（reftool）**：基于 ODrive fibre 框架发现并连接 REF，通过对象树读写 `robot.*` 属性与方法，用于调试与二次开发脚本。

**代码路径**：[`3.Software/CLI-Tool/`](../../3.Software/CLI-Tool/)

## 2. 在总体架构中的位置

对应总体架构上位机侧 **CLI-Tool_reftool**：经 USB Fibre（或串口）访问 REF 协议层。

```mermaid
flowchart LR
  Shell[run_shell_ref_tool] --> FibrePC[fibre_pkg]
  FibrePC -->|USB_or_serial| Proto[REF_Protocols]
  Proto --> Robot[REF_Robot]
```

| 方向 | 邻居模块 | 交互内容 |
|------|----------|----------|
| 下游 | [REF-Protocols](REF-Protocols.md) / [REF-3rdParty](REF-3rdParty.md) | Fibre 对象树 |
| 并列 | [HOST-DummyStudio](HOST-DummyStudio.md) | 同为上位机；Studio 偏 ASCII/示教 |

## 3. 内部结构

```text
CLI-Tool/
├── run_shell.py              # 主入口 argparse
├── run.bat                   # Windows 启动（ipython）
├── fibre/                    # PC 端协议栈与发现
│   ├── discovery.py
│   ├── protocol.py
│   ├── remote_object.py
│   ├── usbbulk_transport.py / serial_transport.py / ...
│   └── shell.py
├── ref_tool/                 # REF 品牌封装（自 odrivetool）
│   ├── shell.py
│   ├── configuration.py
│   ├── code_generator.py
│   ├── utils.py
│   └── version.py
└── _addition/ref_demo.py     # 最小示例
```

| 子路径 | 职责 |
|--------|------|
| `fibre/` | 传输、发现、RemoteObject、通用 shell |
| `ref_tool/` | 设备命名、配置备份恢复、代码生成、绘图工具 |
| `run_shell.py` | CLI 子命令调度 |

## 4. 关键类与入口

| 符号 | 说明 |
|------|------|
| `run_shell.py` | 默认子命令 `shell`；`-p/--path`（默认 `usb`）、`-s/--serial-number` |
| `fibre.discovery.find_any` / `find_all` | 阻塞/回调式发现 |
| `ref_tool.find_any` | 重导出 fibre API |
| `ref_tool.shell.launch_shell` | 交互壳；设备暴露为 `dummy0`…（`branding_short="dummy"`） |

## 5. 核心行为 / 数据流

```text
run_shell.py
  → fibre.find_all(path, serial_number, ...)
       按 usb/serial/tcp/udp 前缀启动发现线程
       读 endpoint JSON → CRC → RemoteObject
  → 交互：dummy0.robot.move_j / joint_1.set_position ...
```

示例（`_addition/ref_demo.py`）：

```python
import ref_tool
my_drive = ref_tool.find_any()
# 或: ref_tool.find_any("serial:COMx")
```

## 6. 对外接口

对用户暴露的是 **Fibre 远程对象**（与固件 `MakeObjTree` 对齐），而非本地 C API。常用路径：

- `robot.set_enable` / `move_j` / `move_l` / `homing` / `joint_n.*`
- 配置：`ref_tool.configuration` 备份/恢复

Fibre 属性全集见 [ARCHITECTURE §4.2](../ARCHITECTURE_CN.md)。

## 7. 配置与约束

- `run_shell.py` / `ref_tool/__init__.py` 中存在指向 `Firmware/fibre/python` 的 `sys.path` 插入；**本仓库实际以 `CLI-Tool/fibre/` 本地包为准**，若路径不存在需注意 import 顺序
- 依赖 Python3、ipython（`run.bat`）、以及 USB 驱动权限（Linux 可用 `ref_tool.version.setup_udev_rules`）
- 部分 argparse 子命令（如 liveplotter）可能注册不完整，以 `run_shell.py` 现有 parser 为准

## 8. 二次开发要点

- **优先修改**：`_addition/` 下写脚本；扩展固件对象树后重新连接即可
- **不宜改动**：随意改 `fibre/protocol.py` 编解码（易与固件不兼容）
- 新增传输：在 `fibre/` 增加 transport 并挂到 discovery

## 9. 相关文档

- [系统架构](../ARCHITECTURE_CN.md) §6.1
- [设计文档索引](README.md)
- [REF-Protocols](REF-Protocols.md) · [REF-3rdParty](REF-3rdParty.md) · [HOST-DummyStudio](HOST-DummyStudio.md)
