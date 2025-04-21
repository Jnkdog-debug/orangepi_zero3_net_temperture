# 🌡️ 家用温湿度采集与展示系统

一个基于香橙派 Zero3 的嵌入式系统项目，支持 LCD 显示、Web UI、Qt 图形界面、触控操作，并可以采集和记录室内外温湿度。

---

## 🔧 项目功能

- 📺 使用 ILI9341 LCD 实时显示温湿度数据
- 🌐 通过 Web 页面查看数据 + 图表
- 📊 本地储存温湿度历史数据
- 📱 支持触摸操作（ft6336）
- 🌈 使用 Qt 创建图形界面
- 🌍 获取外网天气数据（支持 API）
- 📦 自主编写 Linux 驱动模块（学习目的）

---

## 🧱 项目结构

project/ ├── drivers/ # Linux 驱动源码 ├── qt_ui/ # Qt 图形界面 ├── web_server/ # Flask Web 后端 + 页面 ├── data/ # 数据和日志 ├── scripts/ # 启动、初始化脚本 ├── docs/ # 设计文档与笔记 └── README.md # 项目说明文档


---

## 🚀 快速开始

### 环境要求：
- 香橙派 Zero3 + Ubuntu 22.04 (官方镜像)
- 香橙派 zero3 内核6.1.31
- Python3 / Qt / GCC / SPI/I2C 内核支持

### 初始化命令：

```bash
bash scripts/init.sh
bash scripts/start_all.sh
