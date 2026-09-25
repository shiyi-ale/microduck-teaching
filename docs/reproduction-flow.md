# 复现总流程

## 0. 准备目录

建议本地工作区保持三个原仓库并列：

```text
MicroDuck/
  xgoduck_hardware/
  xgoduck_runtime_arduino/
  xgoduck_rl/
```

当前复现路径是：

```text
C:\Users\hangj\Desktop\实验室项目2026\MicroDuck
```

## 1. 硬件复现

先从 `xgoduck_hardware` 开始。

主要文件：

- `bom.xlsx`：整机采购清单。
- `Assembly_Guide.pdf`：装配图。
- `structure/`：3D 打印结构件。
- `PCBA/`：Arduino Uno Q 扩展板设计文件、SMT BOM、贴片坐标、STEP 模型。

关键物料：

- Arduino Uno Q，建议 4GB RAM + 32GB eMMC。
- Robot expansion board。
- Feetech 1910 舵机 15 个。
- AMP 3-pin cable 15 条。
- 8.4 V 18650 电池，建议容量大于 2500 mAh，放电倍率大于 3C。
- M2、M2.5、M3 螺丝和轴承。

结构件里文件名以 `2x_` 开头的需要打印两件。文件名含 `tpu` 的是软材料件，例如脚底和嘴部。

## 2. 扩展板与接线

扩展板叠在 Arduino Uno Q 上，主要承担：

- QMI8658A 六轴 IMU。
- 舵机总线接口。
- 5 V 与 3.3 V 电源。
- 电池接口、开关、DC jack。
- 串口缓冲。

运行时文档中指定：

- 舵机总线连接到 D0/D1，也就是 `Serial1`，1 Mbps。
- QMI8658 连接到 D20/D21，也就是 `Wire`，400 kHz，地址先尝试 `0x6A`，再尝试 `0x6B`。

## 3. 板端运行时部署

在 `xgoduck_runtime_arduino` 中完成。

Uno Q 运行结构：

```text
Linux host: 50 Hz ONNX policy + Web UI :9527
STM32 MCU: 100 Hz servo bus + QMI8658 + Bridge
```

首次运行大致命令：

```bash
python3 tools/prepare-bridge.py
sudo bash tools/router-2m.sh
arduino-app-cli app start "$PWD"
```

其中：

- `prepare-bridge.py` 修改 Arduino 库缓存，让 Bridge/RPClite 适配本项目的数据帧和栈空间。
- `router-2m.sh` 把 Arduino Router 串口速率设到 2 Mbps。
- `arduino-app-cli app start "$PWD"` 启动 App Lab 应用。

Web UI 端口是 `9527`。

## 4. 舵机 ID 与零位校准

第一次装机不要一次接 15 个舵机。先逐个接入舵机，避免多个舵机同 ID 同时响应。

ID 分配：

| ID | 作用 |
| --- | --- |
| 10-14 | 一侧腿部 |
| 20-24 | 另一侧腿部 |
| 30-33 | 脖子和头部 |
| 34 | 嘴部，不属于 14 维策略动作 |

流程：

1. 打开 `/servo.html`。
2. 单个舵机接入，写入目标 ID。
3. 移动到 raw position `2047`。
4. 写入 KP/KD，默认 KP=5，KD=20。
5. 重复直到所有 ID 唯一。
6. 机械装配后打开 `/calibrate.html`，逐关节调零。
7. 零位保存到板端 `data/zero_pos.json`，该文件不进 git。

## 5. 策略运行

runtime 默认开机为 inference only，网络运行但舵机 torque off。

Web UI 控制模式：

| 模式 | 含义 |
| --- | --- |
| Inference only | 只跑策略，不上力 |
| Default pose | 保持 home pose |
| Walk / get up | 启用位置控制和跌倒恢复 |
| Pick | 站立行走状态下运行抓取策略 4 秒 |
| Torque off | 关闭位置控制 |

安全点：

- 页面关闭或后台超过 1 秒会关闭位置控制。
- 250 ms 没有动作命令会停止位置控制。
- IMU 超过 100 ms 无数据会停止位置控制。
- Host feedback 超过 150 ms 会 disarm。

## 6. 训练与 ONNX 导出

训练在 `xgoduck_rl` 中完成。它需要 CUDA GPU、Python 3.12 和 `uv`。

安装：

```bash
uv sync
```

训练平地行走：

```bash
uv run train Mjlab-Velocity-Flat-XgoDuck --env.scene.num-envs 4096
```

导出 ONNX：

```bash
uv run python scripts/export.py Mjlab-Velocity-Flat-XgoDuck \
    --checkpoint-file logs/rsl_rl/xgoduck_velocity/<run>/model_XXXX.pt \
    --onnx-file logs/rsl_rl/xgoduck_velocity/<run>/<run>.onnx
```

导出的 ONNX 需要放到 runtime 的 `python/` 目录，并按 runtime 代码约定命名。

## 7. 复现验收

建议分阶段验收：

1. 结构件装配无干涉。
2. 扩展板供电正常，5 V 和 3.3 V 稳定。
3. 15 个舵机 ID 全部唯一且可读写。
4. IMU 数据正常刷新。
5. Web UI 可打开，状态 API 可读。
6. `Default pose` 能低风险保持姿态。
7. `Walk` 能在保护架或手扶状态下运行。
8. 训练导出的 ONNX 能被 runtime 加载并 warm up。

板端状态检查命令：

```bash
python tools/measure.py --seconds 60
```
