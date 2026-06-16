# DocScanner

一款 Windows 桌面文档扫描软件，利用手机作为摄像头，实现实时文档边缘检测、自动扫描、图像增强和 PDF 生成。

## ✨ 功能特性

### 📱 摄像头连接
- **USB 连接**：通过 ADB 端口转发，延迟低、稳定
- **WiFi 连接**：无线自由，支持局域网连接
- **自动检测**：自动检测设备连接状态

### 🔍 实时边缘检测
- **实时预览**：摄像头画面实时显示
- **文档识别**：自动检测文档四个边缘
- **可视化**：绿色边框 + 红色角点标记
- **稳定性判断**：文档稳定后自动触发扫描

### 📸 扫描模式
- **自动扫描**：文档稳定时自动触发（可开关）
- **手动扫描**：快捷键 `Alt+S` 手动触发
- **批量扫描**：支持连续扫描多页文档

### 🎨 图像增强
- **透视矫正**：自动拉正文档视角
- **灰度化**：转换为灰度图像
- **二值化**：自适应阈值二值化
- **去噪**：去除图像噪点
- **锐化**：增强文字清晰度

### 📄 文档管理
- **新建文档**：`Ctrl+N` 创建新文档
- **页面管理**：支持删除、拖拽排序
- **实时预览**：扫描结果即时预览
- **PDF 命名**：支持自定义文件名

### 📤 导出格式
- **PDF**：标准 PDF 文档
- **图片**：JPG/PNG 单张图片
- **Word**：Microsoft Word 文档

### ⌨️ 快捷键
| 快捷键 | 功能 |
|--------|------|
| `Ctrl+N` | 新建文档（保存上一个为 PDF） |
| `Alt+O` | 切换自动扫描模式 |
| `Alt+S` | 手动扫描 |

### 🎯 视觉反馈
- **扫描成功**：绿色闪光 + 提示音
- **扫描失败**：红色闪光 + 错误音
- **状态栏**：显示连接状态、自动扫描模式、页面数量、最后扫描时间

---

## 🛠️ 技术栈

| 组件 | 技术 | 版本 |
|------|------|------|
| 编程语言 | Python | 3.10+ |
| GUI 框架 | PyQt6 | 6.6+ |
| 计算机视觉 | OpenCV | 4.8+ |
| 数值计算 | NumPy | 1.24+ |
| PDF 生成 | img2pdf | 0.5+ |
| Word 导出 | python-docx | 1.0+ |
| 音频播放 | pygame | 2.5+ |
| 打包工具 | PyInstaller | 6.0+ |
| 手机连接 | ADB (platform-tools) | 34.0+ |

---

## 📦 安装

### 1. 克隆仓库

```bash
git clone https://github.com/Names233/DocScanner.git
cd DocScanner
```

### 2. 安装依赖

```bash
pip install -r requirements.txt
```

### 3. 准备手机端 App

推荐使用 **IP Webcam**（Google Play 免费下载）：

1. 在手机上安装 IP Webcam
2. 打开应用，配置视频分辨率（推荐 1080p）
3. 启动服务器（默认端口 8080）

### 4. 连接手机

**USB 连接：**
1. 手机开启 USB 调试
2. 连接 USB 数据线
3. 运行程序，自动检测设备

**WiFi 连接：**
1. 确保手机和电脑在同一局域网
2. 在程序中输入手机 IP 地址
3. 点击连接

---

## 🚀 使用方法

### 启动程序

```bash
python main.py
```

### 基本流程

1. **连接手机**
   - USB：插入数据线，自动连接
   - WiFi：输入手机 IP，点击连接

2. **新建文档**
   - 按 `Ctrl+N` 创建新文档
   - 输入文档名称（可选）

3. **扫描文档**
   - 将文档放在手机摄像头前
   - 自动模式：等待绿色边框稳定，自动扫描
   - 手动模式：按 `Alt+S` 手动触发

4. **管理页面**
   - 右侧查看扫描结果
   - 支持删除、拖拽排序

5. **导出文档**
   - 点击导出按钮选择格式
   - PDF / 图片 / Word

---

## 📁 项目结构

```
DocScanner/
├── main.py                    # 应用入口
├── app.py                     # 应用主类
├── config.py                  # 配置管理
├── requirements.txt           # 依赖列表
├── build.spec                 # PyInstaller 打包配置
│
├── ui/                        # UI 层
│   ├── main_window.py         # 主窗口
│   ├── camera_view.py         # 摄像头预览组件
│   ├── result_view.py         # 扫描结果组件
│   ├── page_list.py           # 页面列表组件
│   ├── status_bar.py          # 状态栏
│   ├── settings_dialog.py     # 设置对话框
│   ├── export_dialog.py       # 导出对话框
│   └── styles.py              # QSS 样式
│
├── core/                      # 业务逻辑层
│   ├── camera.py              # 摄像头连接管理
│   ├── detector.py            # 文档边缘检测
│   ├── enhancer.py            # 图像增强
│   ├── scanner.py             # 扫描控制逻辑
│   ├── pdf_generator.py       # PDF 生成
│   └── sound_manager.py       # 音效管理
│
├── models/                    # 数据模型
│   ├── document.py            # 文档模型
│   └── page.py                # 页面模型
│
├── utils/                     # 工具类
│   ├── adb_manager.py         # ADB 管理
│   ├── image_utils.py         # 图像工具
│   └── logger.py              # 日志管理
│
├── resources/                 # 资源文件
│   ├── sounds/                # 音效文件
│   ├── icons/                 # 图标
│   └── styles/                # 样式表
│
└── platform-tools/            # ADB 工具（内置）
    ├── adb.exe
    ├── AdbWinApi.dll
    └── AdbWinUsbApi.dll
```

---

## ⚙️ 配置

程序支持配置文件 `config.json`，包含以下配置项：

```json
{
  "connection": {
    "type": "usb",
    "wifi_ip": "192.168.1.100",
    "wifi_port": 8080
  },
  "camera": {
    "resolution": "auto",
    "fps": 30
  },
  "detection": {
    "min_area_ratio": 0.1,
    "stability_threshold": 10,
    "stable_frames": 10,
    "confidence_threshold": 0.8
  },
  "scan": {
    "auto_scan": true,
    "preview_before_save": false,
    "flash_enabled": true,
    "sound_enabled": true
  },
  "export": {
    "default_format": "pdf",
    "output_directory": "./output"
  }
}
```

---

## 🔧 开发

### 环境准备

```bash
# 创建虚拟环境
python -m venv venv
venv\Scripts\activate  # Windows
source venv/bin/activate  # Linux/Mac

# 安装开发依赖
pip install -r requirements.txt
pip install pytest pytest-cov  # 测试工具
```

### 运行测试

```bash
# 运行所有测试
pytest

# 运行带覆盖率的测试
pytest --cov=.
```

### 打包发布

```bash
# 安装 PyInstaller
pip install pyinstaller

# 打包
pyinstaller build.spec

# 输出位置
dist/DocScanner.exe
```

---

## 📋 开发展望

以下功能在当前版本暂不实现，列入未来开发计划：

### 高级参数调整
- 亮度/对比度滑块
- 锐度调整
- 去噪强度调整
- 二值化阈值调整
- 自定义边缘检测参数

### 高级扫描模式
- 批量扫描模式（连续扫描多页）
- 定时扫描
- 运动检测扫描

### 高级图像处理
- OCR 文字识别
- 自动旋转校正
- 多语言支持
- 图像修复

### 云集成
- 云存储同步
- 多设备协作
- 远程扫描

### 高级导出
- 搜索型 PDF（带 OCR 文本层）
- 压缩 PDF
- 水印添加
- 电子签名

---

## 🤝 贡献

欢迎贡献！请遵循以下步骤：

1. Fork 本仓库
2. 创建功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 创建 Pull Request

---

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

---

## 🙏 致谢

- [OpenCV](https://opencv.org/) - 计算机视觉库
- [PyQt6](https://www.riverbankcomputing.com/software/pyqt/) - GUI 框架
- [OpenCV-Document-Scanner](https://github.com/andrewdcampbell/OpenCV-Document-Scanner) - 边缘检测算法参考
- [IP Webcam](https://play.google.com/store/apps/details?id=com.pas.webcam) - 手机端 App

---

## 📞 联系方式

- GitHub: [Names233](https://github.com/Names233)
- 项目链接: [DocScanner](https://github.com/Names233/DocScanner)

---

## ⭐ Star History

如果这个项目对你有帮助，请给个 Star 支持一下！
