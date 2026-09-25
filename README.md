# MicroDuck / XGO Duck Teaching Notes

这个仓库用于教学复现 LuwuDynamics 的 XGO Duck / MicroDuck 系列项目。它不是原始源码的替代品，而是一个面向课程、实验室带教和复现记录的入口仓库。

## 项目由三部分组成

| 模块 | 上游仓库 | 作用 |
| --- | --- | --- |
| 硬件 | https://github.com/LuwuDynamics/xgoduck_hardware | 3D 打印结构件、扩展板、整机 BOM、装配图 |
| 运行时 | https://github.com/LuwuDynamics/xgoduck_runtime_arduino | Arduino Uno Q 上的 Linux host、STM32 firmware、Web UI、ONNX 策略部署 |
| 训练 | https://github.com/LuwuDynamics/xgoduck_rl | MuJoCo/mjlab 强化学习环境、PPO 训练、ONNX 导出 |

## 一句话架构

机器人用 15 个 Feetech 1910 舵机组成双足鸭形结构。Arduino Uno Q 的 Linux 侧以 50 Hz 运行 ONNX 策略和 Web UI，STM32 侧以 100 Hz 处理舵机总线与 QMI8658 IMU。训练通常放在 CUDA 服务器上完成，最终导出 ONNX 模型部署到板端。

## 推荐复现路线

1. 先复现硬件：采购 BOM、打印结构件、制作或外协扩展板。
2. 再复现板端运行时：在 Arduino Uno Q 上启动 runtime，完成舵机 ID 写入、零位校准和 Web UI 控制。
3. 最后复现训练：在服务器上配置 `xgoduck_rl`，训练策略并导出 ONNX。
4. 将训练得到的 ONNX 模型替换到 runtime 的 `python/` 目录中，在实机上低风险测试。

详细流程见：

- [复现总流程](docs/reproduction-flow.md)
- [硬件与物料](docs/hardware-notes.md)
- [运行时部署](docs/runtime-deployment.md)
- [训练与模型导出](docs/training-notes.md)
- [教学安排建议](docs/teaching-plan.md)

## 板卡选择建议

原项目围绕 Arduino Uno Q 设计。BOM 只写了 `Arduino Uno Q`，没有指定 2GB/16GB 或 4GB/32GB 版本。教学复现建议优先使用 4GB RAM + 32GB eMMC 版本，调试空间更宽裕。

Jetson Nano A02 可以作为 Linux 推理侧替代方案，但它没有 Uno Q 内置的 STM32 实时控制侧。若改用 Jetson Nano，需要额外加入 STM32、Teensy、ESP32 或类似 MCU，重写 Jetson 与 MCU 之间的通信和实时控制链路。

## 本仓库不包含的内容

- 不重新分发上游仓库的完整源码。
- 不包含训练生成的 checkpoint、日志或 ONNX 模型。
- 不包含采购链接的价格承诺。价格、库存和板卡版本会变化，采购前需要重新确认。

## 许可证说明

本仓库的教学文档采用 MIT License。上游代码、硬件文件和第三方项目分别遵循其原仓库许可证。
