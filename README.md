# CAN Chassis Tester - 无人驾驶线控底盘测试工具

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

一个全功能的无人驾驶线控底盘测试工具，支持多种CAN工具、内置DBC数据库、支持丰富的控制模式（正弦波、阶跃、斜坡等），可对制动、转向、油门、举升等底盘系统进行完整测试。

## 功能特性

### 🚗 系统支持
- **油门系统** - 加速度控制、速度目标设置
- **制动系统** - 制动力、制动压力控制
- **转向系统** - 转向角、转向力矩控制
- **举升系统** - 举升高度、举升力控制
- **多通道混合** - 同时控制多个系统

### 🔌 CAN工具支持
- **PEAK PCAN** - 高性能CAN接口
- **KVASER** - 工业级CAN接口
- **Virtual CAN** - 虚拟CAN用于开发测试
- 易于扩展支持其他工具

### 📊 控制模式
- **正弦波控制** - 频率、幅值、偏移可配置
- **阶跃控制** - 突变目标值、平台响应
- **斜坡控制** - 线性上升/下降
- **自定义脚本** - 灵活的Python脚本控制

### 📈 数据与报告
- **实时监控** - 实时显示CAN消息和信号值
- **数据采集** - 高效缓冲和存储
- **信号统计** - 最大值、最小值、平均值、标准差
- **报告生成** - 自动生成PDF测试报告
- **数据导出** - CSV格式数据导出
- **波形对比** - 控制指令vs实际响应对比图

### 💾 DBC数据库
- **内置DBC文件** - 常见底盘系统预置
- **自定义DBC** - 支持加载自定义DBC文件
- **信号编解码** - 自动转换原始值<->物理值

## 快速开始

### 安装

```bash
# 克隆仓库
git clone https://github.com/qiaozhi-2023/can-chassis-tester.git
cd can-chassis-tester

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或
venv\Scripts\activate  # Windows

# 安装依赖
pip install -r requirements.txt
```

### 基本使用

```python
from src.can.can_interface import CANInterface
from src.control.signal_generator import SineWaveGenerator

# 初始化CAN接口（使用虚拟CAN进行演示）
can_interface = CANInterface(interface_type='virtual')
can_interface.connect()

# 创建正弦波信号生成器
sine_gen = SineWaveGenerator(
    frequency=1.0,      # 1 Hz
    amplitude=50,       # 50% 油门
    offset=50,          # 50% 偏移
    duration=10         # 10秒
)

# 获取信号值
value = sine_gen.get_value(t=1.5)
```

## 项目结构

```
can-chassis-tester/
├── docs/
│   ├── ARCHITECTURE.md           # 架构设计文档
│   ├── API_REFERENCE.md          # API参考手册
│   ├── USAGE_GUIDE.md            # 详细使用指南
│   └── DBC_GUIDE.md              # DBC文件指南
├── src/
│   ├── can/                      # CAN总线核心
│   ├── dbc/                      # DBC管理
│   ├── control/                  # 控制引擎
│   ├── monitor/                  # 监控与采集
│   ├── report/                   # 报告生成
│   ├── cli/                      # CLI界面
│   └── config/                   # 配置管理
├── examples/                     # 使用示例
├── tests/                        # 单元测试
├── data/                         # 数据文件
├── requirements.txt
├── setup.py
└── README.md
```

## 系统要求

- Python 3.8+
- Linux / macOS / Windows
- CAN接口硬件（可选，支持虚拟CAN进行开发）

## 许可证

MIT License

## 作者

qiaozhi-2023
