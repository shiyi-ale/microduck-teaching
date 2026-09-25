# 训练与模型导出

## 原仓库

训练仓库：

```text
https://github.com/LuwuDynamics/xgoduck_rl
```

本地路径：

```text
C:\Users\hangj\Desktop\实验室项目2026\MicroDuck\xgoduck_rl
```

## 环境要求

- CUDA GPU。
- Python 3.12。
- `uv`。

你前面已经说明 CUDA 暂时不用本机安装，后续放到服务器训练。这是合理路线。本地可以先读代码、准备文档和硬件，训练阶段再迁移到服务器。

## 安装

在服务器上：

```bash
cd xgoduck_rl
uv sync
```

ARM 机器上首次下载 CUDA wheel 可能超时，可设置：

```bash
export UV_HTTP_TIMEOUT=600
```

## 任务列表

可用任务包括：

| 任务 | 用途 |
| --- | --- |
| `Mjlab-Velocity-Flat-XgoDuck` | 平地行走 |
| `Mjlab-Velocity-Rough-XgoDuck` | 粗糙地形行走 |
| `Mjlab-StandUp-Flat-XgoDuck` | 平地起身 |
| `Mjlab-StandUp-Rough-XgoDuck` | 粗糙地形起身 |
| `Mjlab-GroundPick-Flat-XgoDuck` | 平地拾取 |
| `Mjlab-GroundPick-Rough-XgoDuck` | 粗糙地形拾取 |
| `Mjlab-SitStand-Flat-XgoDuck` | 坐站切换 |
| `Mjlab-BallKick-Flat-XgoDuck` | 右脚踢球 |
| `Mjlab-BallKick-Left-Flat-XgoDuck` | 左脚踢球 |
| `Mjlab-Roulade-Flat-XgoDuck` | 翻滚 |

也可以运行：

```bash
uv run list-envs
```

## 训练示例

```bash
uv run train Mjlab-Velocity-Flat-XgoDuck --env.scene.num-envs 4096
```

日志和 checkpoint 通常在：

```text
logs/rsl_rl/<experiment>/
```

## 播放测试

```bash
uv run play Mjlab-Velocity-Flat-XgoDuck
```

指定 checkpoint：

```bash
uv run play Mjlab-Velocity-Flat-XgoDuck \
    --checkpoint-file path/to/model_XXXX.pt
```

## 导出 ONNX

```bash
uv run python scripts/export.py Mjlab-Velocity-Flat-XgoDuck \
    --checkpoint-file logs/rsl_rl/xgoduck_velocity/<run>/model_XXXX.pt \
    --onnx-file logs/rsl_rl/xgoduck_velocity/<run>/<run>.onnx
```

导出时会把 observation normalization bake 进 ONNX 图。

## 部署到实机

根据策略用途重命名并放入 runtime：

```text
xgoduck_runtime_arduino/python/xgoduck_walk.onnx
xgoduck_runtime_arduino/python/xgoduck_getup.onnx
xgoduck_runtime_arduino/python/xgoduck_pick.onnx
```

实机测试顺序：

1. 启动 app。
2. 保持 inference only。
3. 确认模型加载和 warm up 正常。
4. 确认 IMU、舵机状态正常。
5. 切换 default pose。
6. 在保护条件下测试 walk/get up/pick。
