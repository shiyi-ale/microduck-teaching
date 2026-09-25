# 硬件与物料

## 原仓库

硬件仓库：

```text
https://github.com/LuwuDynamics/xgoduck_hardware
```

本地路径：

```text
C:\Users\hangj\Desktop\实验室项目2026\MicroDuck\xgoduck_hardware
```

## BOM 摘要

`bom.xlsx` 中列出的整机物料包括：

| 物料 | 数量 |
| --- | --- |
| Arduino Uno Q | 1 |
| Robot Expansion Board | 1 |
| AMP 3Pin Cable | 15 |
| Feetech 1910 | 15 |
| 18650 battery 8.4V | 1 |
| Countersunk screw M2*6 | 200 |
| Screw M2.5*6 | 6 |
| Screw M3*16 | 4 |
| Bearing 10*15*3 | 2 |
| Bearing 16*22*4 | 11 |

Arduino Uno Q 的 BOM 行没有注明内存/存储版本。教学复现建议选择 4GB RAM + 32GB eMMC 版本。

## 3D 打印

结构件在：

```text
xgoduck_hardware/structure/
```

规则：

- 文件名以 `2x_` 开头：打印两件。
- 文件名带 `left_` 或 `right_`：左右件分别打印。
- 文件名含 `tpu`：建议用 TPU 或对应柔性材料。

## 扩展板

扩展板资料在：

```text
xgoduck_hardware/PCBA/
```

主要文件：

- `ArduinoUnoQ.SchDoc`
- `ArduinoUnoQ.PcbDoc`
- `ArduinoUnoQ.pdf`
- `ArduinoUnoQ.DWG`
- `ArduinoUnoQ.step`
- `BOM.xlsx`
- `Pick Place for ArduinoUnoQ.csv`

扩展板主要功能：

- 给舵机和传感器供电。
- 接入 QMI8658A IMU。
- 引出 15 个舵机的总线连接。
- 与 Arduino Uno Q 堆叠连接。

## 采购与实验室注意事项

- 电池和舵机供电要单独核对电流余量。
- 舵机调 ID 阶段一次只接一个舵机。
- 第一次上电建议串电流表或可调电源限流。
- 第一次行走建议使用保护架、悬吊或手扶。
- 不要在桌面边缘调试有 torque 的机器人。
