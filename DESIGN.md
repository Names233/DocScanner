# DocScanner - 文档扫描软件技术设计文档

## 1. 项目概述

### 1.1 项目目标
开发一款 Windows 桌面文档扫描软件，利用手机作为摄像头，实现实时文档边缘检测、自动扫描、图像增强和 PDF 生成。

### 1.2 核心功能
- 手机摄像头实时预览（USB/WiFi）
- 实时文档边缘检测（绿色边框 + 角点）
- 自动扫描模式（文档稳定时自动触发）
- 手动扫描模式
- 图像增强（灰度、二值化、去噪、锐化、透视矫正）
- PDF 文档管理（新建、扫描、导出）
- 页面管理（删除、排序）

### 1.3 目标用户
需要将纸质文档数字化的个人用户和小型办公室。

---

## 2. 技术栈

| 组件 | 技术选择 | 版本 | 说明 |
|------|----------|------|------|
| 编程语言 | Python | 3.10+ | 主流、生态成熟 |
| GUI 框架 | PyQt6 | 6.6+ | 现代、稳定、功能丰富 |
| 计算机视觉 | OpenCV | 4.8+ | 行业标准 |
| 数值计算 | NumPy | 1.24+ | 图像处理基础 |
| PDF 生成 | img2pdf | 0.5+ | 轻量、保持图片质量 |
| PDF 处理 | reportlab | 4.0+ | 高级 PDF 功能 |
| Word 导出 | python-docx | 1.0+ | Word 文档生成 |
| 音频播放 | pygame | 2.5+ | 提示音播放 |
| 打包工具 | PyInstaller | 6.0+ | exe 打包 |
| 手机连接 | ADB (platform-tools) | 34.0+ | 内置 |

---

## 3. 系统架构

### 3.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DocScanner 应用                              │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   UI 层      │  │   业务层     │  │   数据层     │              │
│  │  (PyQt6)     │  │  (Core)      │  │  (Storage)   │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                 │                  │                      │
│         ▼                 ▼                  ▼                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    核心服务层                                │   │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌──────────┐ │   │
│  │  │ 摄像头服务 │ │ 检测服务   │ │ 增强服务   │ │ PDF 服务 │ │   │
│  │  └────────────┘ └────────────┘ └────────────┘ └──────────┘ │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  手机端                                                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  IP Webcam / 开源 App → MJPEG 流 (HTTP :8080)              │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 模块划分

```
DocScanner/
├── main.py                    # 应用入口
├── app.py                     # 应用主类
├── config.py                  # 配置管理
├── requirements.txt           # 依赖列表
├── build.spec                 # PyInstaller 打包配置
│
├── ui/                        # UI 层
│   ├── __init__.py
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
│   ├── __init__.py
│   ├── camera.py              # 摄像头连接管理
│   ├── detector.py            # 文档边缘检测
│   ├── enhancer.py            # 图像增强
│   ├── scanner.py             # 扫描控制逻辑
│   ├── pdf_generator.py       # PDF 生成
│   └── sound_manager.py       # 音效管理
│
├── models/                    # 数据模型
│   ├── __init__.py
│   ├── document.py            # 文档模型
│   └── page.py                # 页面模型
│
├── utils/                     # 工具类
│   ├── __init__.py
│   ├── adb_manager.py         # ADB 管理
│   ├── image_utils.py         # 图像工具
│   └── logger.py              # 日志管理
│
├── resources/                 # 资源文件
│   ├── sounds/                # 音效文件
│   │   ├── scan_success.wav
│   │   └── scan_fail.wav
│   ├── icons/                 # 图标
│   └── styles/                # 样式表
│
└── platform-tools/            # ADB 工具（内置）
    ├── adb.exe
    ├── AdbWinApi.dll
    └── AdbWinUsbApi.dll
```

---

## 4. 核心模块设计

### 4.1 摄像头模块 (core/camera.py)

#### 4.1.1 连接方式

**USB 连接 (ADB)**
```python
# 1. 检测设备
adb devices

# 2. 端口转发
adb forward tcp:8080 tcp:8080

# 3. 连接 MJPEG 流
http://localhost:8080/video
```

**WiFi 连接**
```python
# 直接连接手机 IP
http://192.168.x.x:8080/video
```

#### 4.1.2 接口设计

```python
class CameraManager:
    def __init__(self):
        self.connection_type = None  # 'usb' or 'wifi'
        self.device_ip = None
        self.is_connected = False
        self.frame = None
    
    def connect_usb(self) -> bool:
        """通过 ADB USB 连接"""
        pass
    
    def connect_wifi(self, ip: str, port: int = 8080) -> bool:
        """通过 WiFi 连接"""
        pass
    
    def disconnect(self):
        """断开连接"""
        pass
    
    def get_frame(self) -> Optional[np.ndarray]:
        """获取当前帧"""
        pass
    
    def get_resolution(self) -> Tuple[int, int]:
        """获取分辨率"""
        pass
    
    def set_resolution(self, width: int, height: int):
        """设置分辨率"""
        pass
```

#### 4.1.3 手机端 App 推荐

**首选：IP Webcam (Google Play)**
- 免费、成熟、稳定
- 支持 MJPEG 流
- 可配置分辨率、帧率
- 无需编译

**备选：Mobile-Webcam (开源)**
- GitHub: https://github.com/soubhagyajit/Mobile-Webcam
- 完全开源，可定制
- 需要自行编译

---

### 4.2 边缘检测模块 (core/detector.py)

#### 4.2.1 检测算法

参考项目：[OpenCV-Document-Scanner](https://github.com/andrewdcampbell/OpenCV-Document-Scanner)

```python
class DocumentDetector:
    def __init__(self):
        self.min_area_ratio = 0.1  # 最小面积占比
        self.stability_threshold = 10  # 稳定性阈值（像素）
        self.stable_frames = 10  # 连续稳定帧数
        self.confidence_threshold = 0.8  # 置信度阈值
    
    def detect_edges(self, frame: np.ndarray) -> Optional[DocumentEdges]:
        """
        检测文档边缘
        
        Returns:
            DocumentEdges: 包含 4 个角点和置信度
        """
        # 1. 灰度化
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        
        # 2. 高斯模糊
        blurred = cv2.GaussianBlur(gray, (5, 5), 0)
        
        # 3. Canny 边缘检测
        edges = cv2.Canny(blurred, 50, 150)
        
        # 4. 膨胀操作（连接断开的边缘）
        kernel = np.ones((3, 3), np.uint8)
        edges = cv2.dilate(edges, kernel, iterations=2)
        
        # 5. 查找轮廓
        contours, _ = cv2.findContours(edges, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
        
        # 6. 筛选最大四边形
        for contour in sorted(contours, key=cv2.contourArea, reverse=True):
            peri = cv2.arcLength(contour, True)
            approx = cv2.approxPolyDP(contour, 0.02 * peri, True)
            
            if len(approx) == 4:
                # 检查面积是否足够大
                area = cv2.contourArea(approx)
                frame_area = frame.shape[0] * frame.shape[1]
                
                if area / frame_area > self.min_area_ratio:
                    return DocumentEdges(
                        corners=approx.reshape(4, 2),
                        confidence=self._calculate_confidence(approx, frame),
                        area=area
                    )
        
        return None
    
    def check_stability(self, edges_history: List[DocumentEdges]) -> bool:
        """检查边缘是否稳定"""
        if len(edges_history) < self.stable_frames:
            return False
        
        recent = edges_history[-self.stable_frames:]
        
        # 计算角点位置变化
        for i in range(1, len(recent)):
            diff = np.abs(recent[i].corners - recent[0].corners)
            if np.max(diff) > self.stability_threshold:
                return False
        
        return True
    
    def draw_edges(self, frame: np.ndarray, edges: DocumentEdges) -> np.ndarray:
        """绘制边缘（绿色边框 + 角点）"""
        result = frame.copy()
        corners = edges.corners.astype(int)
        
        # 绘制绿色边框
        cv2.polylines(result, [corners], True, (0, 255, 0), 2)
        
        # 绘制角点（红色圆点）
        for corner in corners:
            cv2.circle(result, tuple(corner), 8, (0, 0, 255), -1)
        
        return result
```

#### 4.2.2 数据结构

```python
@dataclass
class DocumentEdges:
    corners: np.ndarray      # shape: (4, 2)，四个角点坐标
    confidence: float        # 置信度 0-1
    area: float             # 文档面积
    timestamp: float        # 检测时间戳
```

---

### 4.3 图像增强模块 (core/enhancer.py)

#### 4.4.1 增强管线

```python
class ImageEnhancer:
    def __init__(self):
        self.denoise_strength = 10  # 去噪强度
        self.sharpen_strength = 1.5  # 锐化强度
    
    def enhance(self, image: np.ndarray, corners: np.ndarray) -> np.ndarray:
        """
        完整增强管线
        
        Args:
            image: 原始图像
            corners: 文档四角坐标
        
        Returns:
            增强后的图像
        """
        # 1. 透视矫正
        warped = self._perspective_transform(image, corners)
        
        # 2. 灰度化
        gray = cv2.cvtColor(warped, cv2.COLOR_BGR2GRAY)
        
        # 3. 去噪
        denoised = cv2.fastNlMeansDenoising(gray, None, self.denoise_strength, 7, 21)
        
        # 4. 自适应二值化
        binary = cv2.adaptiveThreshold(
            denoised, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
            cv2.THRESH_BINARY, 11, 2
        )
        
        # 5. 锐化
        sharpened = self._sharpen(binary)
        
        # 6. 转回彩色（可选）
        if len(sharpened.shape) == 2:
            sharpened = cv2.cvtColor(sharpened, cv2.COLOR_GRAY2BGR)
        
        return sharpened
    
    def _perspective_transform(self, image: np.ndarray, corners: np.ndarray) -> np.ndarray:
        """透视变换"""
        # 排序角点：左上、右上、右下、左下
        ordered = self._order_corners(corners)
        
        # 计算目标尺寸
        width = max(
            np.linalg.norm(ordered[1] - ordered[0]),
            np.linalg.norm(ordered[2] - ordered[3])
        )
        height = max(
            np.linalg.norm(ordered[3] - ordered[0]),
            np.linalg.norm(ordered[2] - ordered[1])
        )
        
        # 目标坐标
        dst = np.array([
            [0, 0],
            [width - 1, 0],
            [width - 1, height - 1],
            [0, height - 1]
        ], dtype=np.float32)
        
        # 计算变换矩阵
        M = cv2.getPerspectiveTransform(ordered.astype(np.float32), dst)
        
        # 应用变换
        warped = cv2.warpPerspective(image, M, (int(width), int(height)))
        
        return warped
    
    def _sharpen(self, image: np.ndarray) -> np.ndarray:
        """锐化图像"""
        kernel = np.array([[-1, -1, -1],
                          [-1,  9, -1],
                          [-1, -1, -1]])
        return cv2.filter2D(image, -1, kernel)
    
    def _order_corners(self, corners: np.ndarray) -> np.ndarray:
        """排序角点：左上、右上、右下、左下"""
        # 计算中心点
        center = np.mean(corners, axis=0)
        
        # 按角度排序
        angles = np.arctan2(corners[:, 1] - center[1], corners[:, 0] - center[0])
        sorted_indices = np.argsort(angles)
        
        return corners[sorted_indices]
```

---

### 4.4 PDF 生成模块 (core/pdf_generator.py)

```python
class PDFGenerator:
    def __init__(self):
        self.output_dir = "output"
    
    def create_pdf(self, images: List[np.ndarray], output_path: str) -> bool:
        """
        将图片列表合并为 PDF
        
        Args:
            images: 图片列表（numpy 数组）
            output_path: 输出 PDF 路径
        
        Returns:
            是否成功
        """
        try:
            # 将 numpy 数组转为字节
            image_bytes = []
            for img in images:
                _, buffer = cv2.imencode('.jpg', img, [cv2.IMWRITE_JPEG_QUALITY, 95])
                image_bytes.append(buffer.tobytes())
            
            # 使用 img2pdf 生成 PDF
            with open(output_path, 'wb') as f:
                f.write(img2pdf.convert(image_bytes))
            
            return True
        except Exception as e:
            logger.error(f"PDF 生成失败: {e}")
            return False
    
    def create_word(self, images: List[np.ndarray], output_path: str) -> bool:
        """生成 Word 文档"""
        try:
            doc = Document()
            
            for i, img in enumerate(images):
                # 保存临时图片
                temp_path = f"temp_{i}.jpg"
                cv2.imwrite(temp_path, img)
                
                # 添加到文档
                doc.add_picture(temp_path, width=Inches(6))
                
                # 分页（除最后一页）
                if i < len(images) - 1:
                    doc.add_page_break()
                
                # 删除临时文件
                os.remove(temp_path)
            
            doc.save(output_path)
            return True
        except Exception as e:
            logger.error(f"Word 生成失败: {e}")
            return False
    
    def export_images(self, images: List[np.ndarray], output_dir: str, 
                     format: str = 'jpg') -> List[str]:
        """导出单张图片"""
        paths = []
        for i, img in enumerate(images):
            path = os.path.join(output_dir, f"page_{i+1}.{format}")
            cv2.imwrite(path, img)
            paths.append(path)
        return paths
```

---

### 4.5 文档模型 (models/document.py)

```python
class Document:
    def __init__(self, name: str = ""):
        self.name = name
        self.pages: List[Page] = []
        self.created_at = datetime.now()
        self.modified_at = datetime.now()
    
    def add_page(self, image: np.ndarray, enhanced: bool = True):
        """添加页面"""
        page = Page(
            image=image,
            index=len(self.pages),
            created_at=datetime.now()
        )
        self.pages.append(page)
        self.modified_at = datetime.now()
    
    def remove_page(self, index: int) -> bool:
        """删除页面"""
        if 0 <= index < len(self.pages):
            del self.pages[index]
            # 更新索引
            for i, page in enumerate(self.pages):
                page.index = i
            self.modified_at = datetime.now()
            return True
        return False
    
    def reorder_pages(self, new_order: List[int]) -> bool:
        """重新排序页面"""
        if len(new_order) != len(self.pages):
            return False
        
        self.pages = [self.pages[i] for i in new_order]
        for i, page in enumerate(self.pages):
            page.index = i
        self.modified_at = datetime.now()
        return True
    
    def get_images(self) -> List[np.ndarray]:
        """获取所有页面图像"""
        return [page.image for page in self.pages]
    
    @property
    def page_count(self) -> int:
        return len(self.pages)


@dataclass
class Page:
    image: np.ndarray
    index: int
    created_at: datetime
    enhanced: bool = True
```

---

## 5. UI 设计

### 5.1 主窗口布局

```
┌─────────────────────────────────────────────────────────────────────┐
│  DocScanner                                        [最小化] [关闭] │
├─────────────────────────────────────────────────────────────────────┤
│  [文件]  [设置]  [帮助]                                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────┐ ┆ ┌─────────────────────────────────┐ │
│  │                         │ ┆ │                                 │ │
│  │                         │ ┆ │  文档名称: [scan_20260616    ]  │ │
│  │                         │ ┆ │                                 │ │
│  │    摄像头实时预览        │ ┆ │  ┌─────────────────────────┐   │ │
│  │                         │ ┆ │  │                         │   │ │
│  │    (绿色边框 + 角点)     │ ┆ │  │    最新扫描结果预览      │   │ │
│  │                         │ ┆ │  │                         │   │ │
│  │                         │ ┆ │  │                         │   │ │
│  │                         │ ┆ │  └─────────────────────────┘   │ │
│  │                         │ ┆ │                                 │ │
│  │                         │ ┆ │  页面列表:                      │ │
│  │                         │ ┆ │  ┌───┐ ┌───┐ ┌───┐ ┌───┐      │ │
│  │                         │ ┆ │  │ 1 │ │ 2 │ │ 3 │ │ + │      │ │
│  └─────────────────────────┘ ┆ │  └───┘ └───┘ └───┘ └───┘      │ │
│                              │ │                                 │ │
│           ◄─────拖拽─────►   │ │  [导出 PDF] [导出图片] [导出Word]│ │
│                              │ └─────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────────────┤
│  状态栏: [USB 已连接] [自动扫描: 开] [页面: 5] [最后扫描: 14:30:22]│
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2 状态栏信息

| 位置 | 信息 | 说明 |
|------|------|------|
| 左侧 | 连接状态 | USB 已连接 / WiFi 已连接 / 未连接 |
| 中左 | 自动扫描模式 | 自动扫描: 开 / 关 |
| 中右 | 当前文档页数 | 页面: X |
| 右侧 | 最后扫描时间 | 最后扫描: HH:MM:SS |

### 5.3 快捷键

| 快捷键 | 功能 | 说明 |
|--------|------|------|
| `Ctrl+N` | 新建文档 | 保存当前文档为 PDF，创建新文档 |
| `Alt+O` | 切换自动扫描 | 开/关自动扫描模式 |
| `Alt+S` | 手动扫描 | 手动触发扫描 |

### 5.4 视觉反馈

**扫描成功：**
- 摄像头预览区域短暂闪绿（100ms）
- 播放成功音效
- 状态栏更新

**扫描失败：**
- 摄像头预览区域短暂闪红（100ms）
- 播放失败音效
- 状态栏显示错误

---

## 6. 配置管理

### 6.1 配置文件 (config.json)

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
  "enhancement": {
    "denoise_strength": 10,
    "sharpen_strength": 1.5,
    "output_color": true
  },
  "scan": {
    "auto_scan": true,
    "preview_before_save": false,
    "flash_enabled": true,
    "sound_enabled": true,
    "sound_volume": 0.8
  },
  "export": {
    "default_format": "pdf",
    "output_directory": "./output",
    "pdf_quality": 95
  },
  "ui": {
    "splitter_position": 0.5,
    "window_geometry": null,
    "theme": "light"
  }
}
```

### 6.2 配置管理类

```python
class ConfigManager:
    def __init__(self, config_path: str = "config.json"):
        self.config_path = config_path
        self.config = self.load()
    
    def load(self) -> dict:
        """加载配置"""
        if os.path.exists(self.config_path):
            with open(self.config_path, 'r', encoding='utf-8') as f:
                return json.load(f)
        return self.get_default()
    
    def save(self):
        """保存配置"""
        with open(self.config_path, 'w', encoding='utf-8') as f:
            json.dump(self.config, f, indent=2, ensure_ascii=False)
    
    def get(self, key: str, default=None):
        """获取配置值"""
        keys = key.split('.')
        value = self.config
        for k in keys:
            if isinstance(value, dict):
                value = value.get(k)
            else:
                return default
        return value if value is not None else default
    
    def set(self, key: str, value):
        """设置配置值"""
        keys = key.split('.')
        config = self.config
        for k in keys[:-1]:
            if k not in config:
                config[k] = {}
            config = config[k]
        config[keys[-1]] = value
        self.save()
    
    def get_default(self) -> dict:
        """获取默认配置"""
        return {
            "connection": {"type": "usb"},
            "camera": {"resolution": "auto", "fps": 30},
            "detection": {
                "min_area_ratio": 0.1,
                "stability_threshold": 10,
                "stable_frames": 10,
                "confidence_threshold": 0.8
            },
            "scan": {
                "auto_scan": True,
                "preview_before_save": False,
                "flash_enabled": True,
                "sound_enabled": True
            },
            "export": {
                "default_format": "pdf",
                "output_directory": "./output"
            }
        }
```

---

## 7. 日志管理

### 7.1 日志配置

```python
import logging
from logging.handlers import RotatingFileHandler

def setup_logger(name: str = "DocScanner", log_file: str = "logs/app.log"):
    """配置日志系统"""
    logger = logging.getLogger(name)
    logger.setLevel(logging.DEBUG)
    
    # 文件处理器（轮转日志）
    file_handler = RotatingFileHandler(
        log_file, maxBytes=10*1024*1024, backupCount=5, encoding='utf-8'
    )
    file_handler.setLevel(logging.DEBUG)
    
    # 控制台处理器
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)
    
    # 格式
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    file_handler.setFormatter(formatter)
    console_handler.setFormatter(formatter)
    
    logger.addHandler(file_handler)
    logger.addHandler(console_handler)
    
    return logger
```

---

## 8. 打包与发布

### 8.1 PyInstaller 配置 (build.spec)

```python
# -*- mode: python ; coding: utf-8 -*-

block_cipher = None

a = Analysis(
    ['main.py'],
    pathex=[],
    binaries=[
        ('platform-tools/adb.exe', 'platform-tools'),
        ('platform-tools/AdbWinApi.dll', 'platform-tools'),
        ('platform-tools/AdbWinUsbApi.dll', 'platform-tools'),
    ],
    datas=[
        ('resources/sounds/*', 'resources/sounds'),
        ('resources/icons/*', 'resources/icons'),
        ('resources/styles/*', 'resources/styles'),
    ],
    hiddenimports=[],
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[],
    win_no_prefer_redirects=False,
    win_private_assemblies=False,
    cipher=block_cipher,
    noarchive=False,
)

pyz = PYZ(a.pure, a.zipped_data, cipher=block_cipher)

exe = EXE(
    pyz,
    a.scripts,
    a.binaries,
    a.zipfiles,
    a.datas,
    [],
    name='DocScanner',
    debug=False,
    bootloader_ignore_signals=False,
    strip=False,
    upx=True,
    upx_exclude=[],
    runtime_tmpdir=None,
    console=False,
    disable_windowed_traceback=False,
    argv_emulation=False,
    target_arch=None,
    codesign_identity=None,
    entitlements_file=None,
    icon='resources/icons/app.ico',
)
```

### 8.2 打包命令

```bash
# 安装依赖
pip install -r requirements.txt

# 打包
pyinstaller build.spec

# 输出目录
dist/DocScanner.exe
```

---

## 9. 依赖列表 (requirements.txt)

```
PyQt6>=6.6.0
opencv-python>=4.8.0
numpy>=1.24.0
img2pdf>=0.5.0
reportlab>=4.0.0
python-docx>=1.0.0
pygame>=2.5.0
pyinstaller>=6.0.0
```

---

## 10. 开发计划

### 10.1 里程碑

| 阶段 | 时间 | 目标 |
|------|------|------|
| M1 | 第 1 周 | 项目搭建、摄像头连接 |
| M2 | 第 2 周 | 边缘检测、实时预览 |
| M3 | 第 3 周 | 图像增强、PDF 生成 |
| M4 | 第 4 周 | UI 完善、快捷键、音效 |
| M5 | 第 5 周 | 测试、打包、发布 |

### 10.2 详细任务

**M1: 项目搭建**
- [x] 创建项目结构
- [ ] 实现配置管理
- [ ] 实现日志系统
- [ ] 实现 ADB 管理
- [ ] 实现摄像头连接（USB/WiFi）
- [ ] 创建主窗口框架

**M2: 边缘检测**
- [ ] 实现边缘检测算法
- [ ] 实现稳定性判断
- [ ] 实现边框绘制
- [ ] 集成到摄像头预览
- [ ] 实现自动扫描逻辑

**M3: 图像增强与 PDF**
- [ ] 实现透视变换
- [ ] 实现图像增强管线
- [ ] 实现 PDF 生成
- [ ] 实现 Word 导出
- [ ] 实现图片导出

**M4: UI 完善**
- [ ] 实现页面列表（拖拽排序）
- [ ] 实现导出对话框
- [ ] 实现设置对话框
- [ ] 实现快捷键系统
- [ ] 实现音效系统
- [ ] 实现视觉反馈

**M5: 测试与发布**
- [ ] 单元测试
- [ ] 集成测试
- [ ] 性能优化
- [ ] 打包为 exe
- [ ] 编写用户文档

---

## 11. 开发展望

以下功能在当前版本暂不实现，列入未来开发计划：

### 11.1 高级参数调整
- 亮度/对比度滑块
- 锐度调整
- 去噪强度调整
- 二值化阈值调整
- 自定义边缘检测参数

### 11.2 高级扫描模式
- 批量扫描模式（连续扫描多页）
- 定时扫描
- 运动检测扫描

### 11.3 高级图像处理
- OCR 文字识别
- 自动旋转校正
- 多语言支持
- 图像修复

### 11.4 云集成
- 云存储同步
- 多设备协作
- 远程扫描

### 11.5 高级导出
- 搜索型 PDF（带 OCR 文本层）
- 压缩 PDF
- 水印添加
- 电子签名

---

## 12. 许可证

MIT License

---

## 13. 参考资源

| 资源 | 链接 | 用途 |
|------|------|------|
| OpenCV 文档 | https://docs.opencv.org/ | 图像处理 |
| PyQt6 文档 | https://www.riverbankcomputing.com/static/Docs/PyQt6/ | GUI 开发 |
| OpenCV-Document-Scanner | https://github.com/andrewdcampbell/OpenCV-Document-Scanner | 边缘检测算法 |
| img2pdf 文档 | https://pypi.org/project/img2pdf/ | PDF 生成 |
| IP Webcam | Google Play | 手机端 App |
