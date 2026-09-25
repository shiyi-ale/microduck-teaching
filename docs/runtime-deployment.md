# 运行时部署

## 原仓库

运行时仓库：

```text
https://github.com/LuwuDynamics/xgoduck_runtime_arduino
```

本地路径：

```text
C:\Users\hangj\Desktop\实验室项目2026\MicroDuck\xgoduck_runtime_arduino
```

## 运行结构

Arduino Uno Q 被项目当作两个计算单元：

| 单元 | 频率 | 工作 |
| --- | --- | --- |
| Qualcomm Linux | 50 Hz | ONNX Runtime、Web UI、策略切换 |
| STM32U585 / Zephyr | 100 Hz | 舵机总线、QMI8658、Bridge 状态回传 |

Host 和 MCU 之间通过 Arduino Bridge 通信，项目将 router 串口速率设为 2 Mbps。

## 依赖

Python 依赖：

```text
numpy==2.2.6
onnxruntime==1.22.1
```

Arduino Zephyr profile：

```text
arduino:zephyr:unoq
Arduino_RouterBridge 0.4.3
Arduino_RPClite 0.3.1
MsgPack 0.4.2
ArxContainer 0.7.0
ArxTypeTraits 0.3.2
DebugLog 0.8.4
```

## 启动流程

在 Uno Q 上进入 `xgoduck_runtime_arduino`：

```bash
python3 tools/prepare-bridge.py
sudo bash tools/router-2m.sh
arduino-app-cli app start "$PWD"
```

启动后访问：

```text
http://<uno-q-ip>:9527
```

## 模型文件

runtime 期望在 `python/` 目录中找到：

```text
xgoduck_walk.onnx
xgoduck_getup.onnx
xgoduck_pick.onnx
```

模型输入输出：

- 输入 observation: `[1, 61]`
- 输出 action: `[1, 14]`

嘴部 ID 34 不在 14 维动作里，由 Web UI 控制。

## 状态检查

只读检查，不会启用电机：

```bash
python tools/measure.py --seconds 60
```

可加 `--output` 保存 JSON 记录。

## 常见风险

- `router-2m.sh` 会改变 Uno Q 全局 Arduino Router 速率。跑其他默认 115200 的 App 前，需要恢复或移走 drop-in 配置。
- `prepare-bridge.py` 修改的是板端 Arduino 库缓存。库缓存清理后需要重新执行。
- `data/zero_pos.json` 是实机校准结果，不应提交到 git。
- ONNX 模型替换后要先 inference only，再 torque on。
