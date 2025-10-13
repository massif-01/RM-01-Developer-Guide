# RM-01 开发者指南 | RM-01 Developer Guide

[![Language: Multilingual](https://img.shields.io/badge/Language-Multilingual-blue.svg)](https://github.com/massif-01/RM-01-Developer-Guide)
[![Device: RM-01](https://img.shields.io/badge/Device-RM%2D01-orange.svg)](https://github.com/massif-01/RM-01-Developer-Guide)
[![Documentation: Complete](https://img.shields.io/badge/Documentation-Complete-green.svg)](https://github.com/massif-01/RM-01-Developer-Guide)

欢迎来到 RM-01 开发者指南！本仓库提供 RM-01 高性能便携 AI 设备的完整多语言开发者文档。

Welcome to the RM-01 Developer Guide! This repository provides comprehensive multilingual developer documentation for the RM-01 high-performance portable AI device.

---

## 🚀 关于 RM-01 | About RM-01

RM-01 是一款革命性的便携式 AI 推理设备，集成了三个独立模组：**推理模组**、**应用模组** 和 **带外管理模组**。通过 USB Type-C 连接即可自动创建本地网络环境，让您能够轻松部署、测试和运行 AI 模型。

RM-01 is a revolutionary portable AI inference device featuring three integrated modules: **Inference Module**, **Application Module**, and **Out-of-Band Management Module**. Simply connect via USB Type-C to automatically create a local network environment for easy AI model deployment, testing, and execution.

### 🔧 核心组件 | Core Components

| 组件 Component | 规格 Specifications | IP 地址 IP Address |
|---|---|---|
| **应用模组** <br> Application Module | Intel Core i3-N305, 最高 24GB LPDDR5 <br> Intel Core i3-N305, up to 24GB LPDDR5 | `10.10.99.99` |
| **推理模组** <br> Inference Module | 最高 128GB 显存, 2,070 TFLOPS (FP4) <br> Up to 128GB VRAM, 2,070 TFLOPS (FP4) | `10.10.99.98` |
| **管理模组** <br> Management Module | RobOS 监控面板 <br> RobOS Monitoring Dashboard | `10.10.99.97` |
| **用户主机** <br> User Host | 自动分配 Auto-assigned | `10.10.99.100` |

### ✨ 主要特性 | Key Features

- 🔌 **即插即用**: USB Type-C 连接自动配置网络环境 | **Plug & Play**: Auto network setup via USB Type-C
- 🧠 **AI 推理优化**: 预装 vLLM 和 TEI 框架 | **AI-Optimized**: Pre-installed vLLM and TEI frameworks  
- 📊 **实时监控**: RobOS 系统性能监控面板 | **Real-time Monitoring**: RobOS system performance dashboard
- 🔧 **灵活部署**: 支持 Docker 容器化应用 | **Flexible Deployment**: Docker containerized applications
- 💾 **模型管理**: CFexpress Type-B 存储卡模型加载 | **Model Management**: CFexpress Type-B storage for models

---

## 📚 文档语言版本 | Documentation Languages

选择您的语言版本以获取详细的开发者指南：

Choose your language for detailed developer documentation:

| 语言 Language | 文档链接 Documentation Link | 状态 Status |
|---|---|---|
| 🇬🇧 English | [RM-01 Developer Guide](./RM-01%20Developer%20Guide.md) | ✅ 完整 Complete |
| 🇨🇳 简体中文 | [RM-01 开发者使用指南](./RM-01%20开发者使用指南.md) | ✅ 完整 Complete |
| 🇯🇵 日本語 | [RM-01 開発者ガイド](./RM-01%20開発者ガイド.md) | ✅ 完整 Complete |
| 🇰🇷 한국어 | [RM-01 개발자 가이드](./RM-01%20개발자%20가이드.md) | ✅ 완료 Complete |
| 🇫🇷 Français | [Guide du Développeur RM-01](./Guide%20du%20Développeur%20RM-01.md) | ✅ 完整 Complete |
| 🇮🇹 Italiano | [Guida per Sviluppatori RM-01](./Guida%20per%20Sviluppatori%20RM-01.md) | ✅ 完整 Complete |
| 🇪🇸 Español | [Guía del Desarrollador RM-01](./Guía%20del%20Desarrollador%20RM-01.md) | ✅ 完整 Complete |

---

## 🛠️ 快速开始 | Quick Start

### 1. 连接设备 | Connect Device
```bash
# 使用 USB Type-C 数据线连接 RM-01 到您的电脑
# Connect RM-01 to your computer using USB Type-C cable
```

### 2. 网络验证 | Network Verification
```bash
# 验证网络连接 | Verify network connection
ping 10.10.99.99  # 应用模组 | Application Module
ping 10.10.99.98  # 推理模组 | Inference Module  
ping 10.10.99.97  # 管理模组 | Management Module
```

### 3. SSH 访问 | SSH Access
```bash
# 连接到应用模组 | Connect to Application Module
ssh rm01@10.10.99.99
# 默认密码 | Default password: rm01
```

### 4. Web 界面 | Web Interface
- **Open WebUI**: http://10.10.99.99 (模型调试 | Model debugging)
- **RobOS 监控**: http://10.10.99.97 (系统监控 | System monitoring)
- **vLLM API**: http://10.10.99.98:58000/v1/chat/completions

---

## 🔧 开发工作流 | Development Workflow

### 模型部署 | Model Deployment

1. **自动模式** | **Automatic Mode**
   ```bash
   # 将模型文件直接放入 CFexpress 存储卡的 auto/llm/ 目录
   # Place model files directly in auto/llm/ directory of CFexpress card
   ```

2. **手动模式** | **Manual Mode** 
   ```yaml
   # 创建 dev/llm/llm_run.yaml 配置文件
   # Create dev/llm/llm_run.yaml configuration file
   model: /home/rm01/models/dev/llm/your-model
   port: 58000
   gpu_memory_utilization: 0.85
   max_model_len: 24576
   ```

### 应用部署 | Application Deployment
```bash
# 上传 Docker 镜像到应用模组
# Upload Docker image to Application Module
docker save your-app:latest | ssh rm01@10.10.99.99 'docker load'
```

---

## 📋 系统要求 | System Requirements

### 主机系统 | Host System
- **操作系统**: macOS 10.15+, Windows 10+, Linux (Ubuntu 18.04+)
- **接口**: USB Type-C 数据接口
- **网络**: 支持 USB Ethernet 适配器

### CFexpress 存储卡 | CFexpress Storage Card
- **类型**: CFexpress Type-B
- **容量**: 建议 256GB+ (推荐 1TB+ 用于大模型)
- **速度**: PCIe 3.0 x2 或更高

---

## 🔒 安全须知 | Security Notes

⚠️ **重要**: 首次 SSH 登录后请立即修改默认密码
⚠️ **Important**: Change default password immediately after first SSH login

```bash
# 修改应用模组密码 | Change Application Module password
passwd
```

---

## 🤝 贡献 | Contributing

欢迎提交问题和改进建议！请确保遵循我们的贡献指南。

We welcome issues and improvement suggestions! Please follow our contribution guidelines.

### 问题报告 | Issue Reporting
- 🐛 [报告 Bug | Report Bug](https://github.com/massif-01/RM-01-Developer-Guide/issues)
- 💡 [功能请求 | Feature Request](https://github.com/massif-01/RM-01-Developer-Guide/issues)
- 📖 [文档改进 | Documentation Improvement](https://github.com/massif-01/RM-01-Developer-Guide/issues)

---

## 📄 许可证 | License

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 📞 支持 | Support

- 📚 **文档**: [开发者指南](./RM-01%20开发者使用指南.md) | [Developer Guide](./RM-01%20Developer%20Guide.md)
- 🌐 **官网**: [massif.ai](https://massif.ai)
- 📧 **邮箱**: support@massif.ai

---

<div align="center">
  <b>🚀 开始您的 RM-01 AI 开发之旅！| Start Your RM-01 AI Development Journey! 🚀</b>
</div>
