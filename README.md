# 车牌识别系统（License Plate Recognition System）

> 基于 **PyQt5 + Ultralytics YOLO + PaddleOCR** 的 Windows 桌面车牌识别系统
> 支持 18 种中国车牌类型、4 种输入源、12 项运行时可调参数、视频自动去重

---

## 📑 目录

- [功能特性](#-功能特性)
- [项目结构](#-项目结构)
- [环境要求](#-环境要求)
- [安装步骤](#-安装步骤)
- [首次运行](#-首次运行)
- [参数设置](#-参数设置)
- [支持的车牌类型](#-支持的车牌类型19-种)
- [结果保存](#-结果保存)
- [常见问题](#-常见问题-faq)
- [开发者说明](#-开发者说明)

---

## ✨ 功能特性

| 功能 | 说明 |
|---|---|
| 🎯 **四种输入源** | 单张图片 / 图片文件夹 / 视频文件 / 实时摄像头 |
| 🚗 **全流程车牌识别** | YOLO 定位 → 裁剪 → HSV 判色 → PaddleOCR 识别号码 |
| 🏷️ **19 类车牌分类** | 普通蓝牌 / 新能源 / 黄牌 / 军/武警 / 警 / 使馆 / 港澳 / 多国等 |
| 🎨 **智能颜色判定** | HSV 多区间累加；支持红色跨色环；支持大型新能源黄绿渐变 |
| 📐 **双层车牌支持** | 自动识别单双层（宽高比阈值），上下两半分别 OCR 后拼接 |
| 🌗 **深色车牌优化** | 黑底白字（使馆/港澳/特殊黑牌）自动反色 OCR，取最优结果 |
| ⏯️ **过程控制** | 开始 / 暂停 / 继续 / 停止，所有耗时操作在 QThread 中，UI 不卡 |
| ⚙️ **12 项可调参数** | YOLO 阈值、长度限制、跳帧、去重等运行时可调，立即生效 |
| 🚫 **无效结果过滤** | 自动跳过未识别、全中文、乱码等无效 OCR 结果 |
| 🔁 **视频自动去重** | 同一车牌在时间窗内只记录一次，解决停车场景重复计数问题 |
| 📊 **结果导出** | 带框图像 + 分离车牌小图 + Excel 汇总表（含统计表） |
| 🔬 **OCR 自检工具** | 一键诊断排查 OCR 失败原因，打印完整中间步骤 |
| 🌐 **完善中文支持** | 中文 UI / 中文路径 / PIL 绘制中文标签防乱码 |

---

## 📁 项目结构

```
LicensePlateSystem/
├── main.py              # 主程序入口（UI 层 + 参数设置对话框）
├── detector.py          # 核心检测逻辑（YOLO + 颜色判定 + PaddleOCR）
├── workers.py           # QThread 工作线程（模型加载 / 检测循环）
├── result_manager.py    # 结果管理（累积 / 去重 / Excel 导出）
├── config.py            # 全局配置（路径、颜色范围、默认参数）
├── fix_paddle.bat       # PaddlePaddle 版本修复脚本（一键）
├── requirements.txt     # 依赖清单
└── README.md            # 本文件
```

### 分层架构

- **UI 层** (`main.py`) — 只负责渲染与交互，不含算法
- **业务层** (`detector.py`、`result_manager.py`) — 封装算法与数据
- **线程层** (`workers.py`) — 用 QThread 异步化所有耗时操作
- **配置层** (`config.py`) — 集中管理常量与默认值

---

## 💻 环境要求

- **操作系统**：Windows 10 / 11（x86_64）
- **Python**：3.10（推荐，实测 Anaconda + Python 3.10 稳定）
- **内存**：≥ 4 GB
- **显卡**：可选。NVIDIA GPU 会显著加快 YOLO；PaddleOCR 支持 GPU 推理

---

## 🚀 安装步骤

### 1. 创建 conda 环境

```bash
conda create -n yolo python=3.10 -y
conda activate yolo
```

### 2. 安装依赖

```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

⚠️ **重要版本约束**

`requirements.txt` 已经把 `paddlepaddle` 固定在 **2.6.2**、`paddleocr` 固定在 **2.7.0.3**，**不要升级**。原因：

- **PaddlePaddle 3.3.0** 在 CPU 模式下有 PIR + oneDNN 的 `ConvertPirAttribute2RuntimeAttribute not support` 官方回归 bug（[issue #77340](https://github.com/PaddlePaddle/Paddle/issues/77340)）
- **PaddleOCR 3.x** 的 API 与 2.x 不兼容、模型更大、初始化更慢，对车牌这种窄场景无明显收益

如果你已经装了 3.x 版本遇到上述错误，双击 `fix_paddle.bat` 一键降级。

### 3. GPU 用户额外步骤

如果要用 NVIDIA GPU 加速 PaddleOCR：

```bash
pip uninstall -y paddlepaddle
pip install paddlepaddle-gpu==2.6.2
```

注意：你电脑的 CUDA 版本要和 PaddlePaddle 兼容。

---

## 🎬 首次运行

### 1. 修改配置路径

编辑 `config.py`，把三个默认路径改成你自己的：

```python
DEFAULT_MODEL_PATH  = r"E:\DeepLearning\YOLO-Chinese_License_Plate_Detection_System\ultralytics-main\weights\best.pt"
DEFAULT_SAVE_DIR    = r"E:\DeepLearning\YOLO-Chinese_License_Plate_Detection_System\ultralytics-main\runs\save_results"
CHINESE_FONT_PATH   = r"E:\DeepLearning\YOLO-Chinese_License_Plate_Detection_System\ultralytics-main\Font\DroidSansFallback.ttf"
```

### 2. 启动程序

```bash
python main.py
```

### 3. 标准使用流程

| 步骤 | 操作 | 说明 |
|---|---|---|
| 1 | 菜单【模型 → 选择模型】 | 指定你的 `best.pt` 文件（默认路径已填好） |
| 2 | 菜单【模型 → 初始化模型】 | 等待 30-60 秒，首次会下载 PaddleOCR 模型（约 18MB） |
| 3 | 【输入源 → 选择图片/文件夹/视频/摄像头】 | 选择你要检测的内容 |
| 4 | 工具栏【▶ 开始检测】 | 点了才开始检测，**不会自动运行** |
| 5 | 工具栏【⏸ 暂停/继续】/【⏹ 停止】 | 随时控制检测进度 |
| 6 | 工具栏【💾 保存结果】 | 导出带框图像 + 车牌小图 + Excel |

### 4. 首次建议：运行 OCR 自检

菜单【模型 → 🔬 诊断 OCR】，用合成的"京A12345"蓝底白字图跑一次 OCR 全流程，日志区会输出：
- PaddleOCR / PaddlePaddle 版本
- OCR 调用是否成功
- 返回数据结构详情
- 最终识别结果

如果自检不通过，说明 OCR 环境有问题，请先解决再用真实车牌测试。

---

## ⚙️ 参数设置

点击工具栏【⚙ 参数】或菜单【设置 → 参数设置】打开调参对话框。所有 12 个参数**修改后立即生效**，无需重启或重新加载模型。

### 参数一览

| 分组 | 参数 | 默认 | 说明 |
|---|---|---|---|
| **YOLO 检测** | YOLO 置信度阈值 | 0.50 | 值越高只保留"非常像车牌"的框；低阈值召回更多但可能误检 |
| | YOLO IOU 阈值 | 0.45 | NMS 去重阈值，值越低去重越严格 |
| **号码有效性过滤** | 跳过无效车牌 | ✅ | 总开关。开启后下列规则生效 |
| | 车牌号最短长度 | 5 | 低于此长度视为无效 |
| | 车牌号最长长度 | 10 | 高于此长度视为无效 |
| | 必须含字母/数字 | ✅ | 过滤纯中文的 OCR 乱码 |
| | **必须含汉字+字母+数字**（严格） | ❌ | 同时要求三类字符都有。适合普通民用车牌，但会拒绝军牌、多国牌等 |
| | 允许乱码字符占比 | 0.20 | 非字母/数字/汉字字符占比超过此值判为乱码 |
| **视频 / 去重** | **视频跳帧** | 2 | 视频每 N 帧处理一次。CPU 慢/长视频可调到 5-10 |
| | **摄像头跳帧** | 3 | 摄像头每 N 帧处理一次 |
| | **结果去重** | ✅ | 同号码在时间窗内只记一次，避免视频重复记录 |
| | **去重时间窗** | 5.0 秒 | 过期后重新可以记录（如车辆 2 分钟后返回场景） |

### 无效车牌过滤规则

勾选"跳过无效车牌"后，满足以下**任一条件**的结果会被直接丢弃（不显示、不计入统计、不保存）：

1. 空字符串或等于 `'未识别'`、`'未初始化'`、`'未知'`
2. 长度不在 `[min_length, max_length]` 范围内
3. `require_alphanumeric=True` 时：不含任何 A-Z/0-9（纯中文认定为 OCR 出错）
4. `require_cn_letter_num=True` 时（严格模式）：没有同时含有汉字、字母、数字三种字符
5. 非字母/数字/汉字的字符占比 > `max_garbage_ratio`（如 `?￥%&#` 等乱码）

### 视频去重逻辑

维护一个 `{车牌号: 最后一次看到的时间}` 字典：

- 新检测进来时查字典，若号码在 `dedupe_window_sec` 秒内出现过 → **跳过**
- 字典条目自动清理过期项
- **UI 实时显示**仍每帧更新（你能看到车在画面里），**表格/统计**只记录去重后的

开始新一轮检测时自动清空字典，避免上段视频的号码污染新视频。

### 调参场景建议

| 场景 | 建议参数 |
|---|---|
| 白天清晰图片 | 默认即可 |
| 夜间 / 雾天 / 模糊 | 置信度 0.3、乱码占比 0.3 |
| 长视频（CPU 机） | 视频跳帧 5-10 |
| 固定路口 24h 监控 | 去重窗口 60-120 秒（同车 2 分钟内只记一次） |
| 多辆车快速驶过 | 去重窗口 2-3 秒，视频跳帧 1 |
| 只要普通民用车牌 | 开启「必须含汉字+字母+数字」严格模式 |
| 包含军牌/使馆等 | 关闭严格模式，保留宽松过滤即可 |

---

## 🚗 支持的车牌类型（18 种）

根据**颜色 + 布局 + 特殊字符**三维度判定：

| 类别 | 典型号码 | 判定依据 |
|---|---|---|
| 普通蓝牌 | 京B·02128 | 蓝色 |
| 单层黄牌 | 京A·F0236 | 黄色 + 单层 |
| 双层黄牌 | 京A·F0236 | 黄色 + 双层 |  
| 单层军牌 | KA·02128 | 白色 + 字母开头 + 无汉字 + 单层 |
| 双层军牌 | KA·30520 | 同上 + 双层 |   
| 单层武警车牌 | WJ²²01888 | 含 `WJ` + 单层 |
| 双层武警车牌 | WJ 皖 12069 | 含 `WJ` + 双层 |     
| 小型新能源车牌 | 京A·A12347 | 绿色 + 单层 |
| 大型新能源车牌 | 京A·A12347 | 绿色（渐变黄绿）+ 双层 |
| 警牌 | 京A0006警 | 含 `警` |
| 使馆车牌 | 使133·001 | 含 `使` |
| 领事馆车牌 | 沪A·0023领 | 含 `领` |
| 教练车牌 | 黑A·1234学 | 含 `学` |
| 挂车车牌 | —·—挂 | 含 `挂` |
| 港澳出入境(港) | 粤Z·F023港 | 含 `港` |
| 港澳出入境(澳) | 粤Z·F023澳 | 含 `澳` |
| 民航车牌 | 民航·M2093 | 含 `民航` |
| 特殊黑牌 | 京A·I9598 | 黑色 |


**重要说明**：车牌类型识别只是 OCR 识别后的分类逻辑。前提是你的 YOLO `best.pt` 能够**检测出这些类型的 bounding box**。如果模型只用普通蓝牌训练过，其他类型根本框不出来 —— 需要用 [CCPD2020](https://github.com/detectRecog/CCPD) 或 [CRPD](https://github.com/yxgong0/CRPD) 等多类型数据集重训。

---

## 💾 结果保存

点击工具栏【💾 保存结果】后，在 `DEFAULT_SAVE_DIR` 下生成时间戳子目录：

```
E:\...\runs\save_results\20260421_155032\
├── images/                 带检测框的完整原图
│   ├── image1.jpg
│   ├── video_frame_60.jpg
│   └── ...
├── plates/                 分离出的车牌小图
│   ├── 0001_image1_京A12345.jpg
│   ├── 0002_image1_粤B_02128.jpg
│   └── ...
└── 车牌信息表.xlsx         Excel 汇总（两个工作表）
```

Excel 内容：

**工作表 1「车牌信息表」**

| 序号 | 检测时间 | 来源 | 车牌号码 | 车牌类型 | 车牌颜色 | 是否双层 | 置信度 |
|---|---|---|---|---|---|---|---|
| 1 | 2026-04-21 15:50:32 | image1.jpg | 京A12345 | 普通蓝牌 | 蓝色 | 否 | 0.952 |

**工作表 2「统计信息」**

| 指标 | 数值 |
|---|---|
| 检测总数 | 15 |
| 唯一车牌数 | 12 |
| 蓝色 车牌数 | 8 |
| 黄色 车牌数 | 3 |
| 绿色 车牌数 | 4 |

---

## ❓ 常见问题（FAQ）

### Q1: 初始化模型报错 `ConvertPirAttribute2RuntimeAttribute not support`

**原因**：PaddlePaddle 3.3.0 在 CPU 模式下的官方确认 bug（[issue #77340](https://github.com/PaddlePaddle/Paddle/issues/77340)）。

**解决**：双击项目目录下的 `fix_paddle.bat`，或手动执行：

```bash
pip uninstall -y paddlepaddle paddleocr paddlex
pip install paddlepaddle==2.6.2
pip install paddleocr==2.7.0.3
```

### Q2: 报错 `'PaddleOCR' object has no attribute 'predict'`

**原因**：PaddleOCR 2.x 使用 `.ocr()` 方法，3.x 才有 `.predict()`。这是版本误判问题。

**解决**：确认你装的是 `paddleocr==2.7.0.3`（不是 3.x），重启程序。代码会根据 `paddleocr.__version__` 主版本自动选择 API 分支。

### Q3: 检测到车牌但号码一直显示"未识别"

**排查步骤**：
1. 菜单【模型 → 🔬 诊断 OCR】看 OCR 流水线自检是否通过
2. 【⚙ 参数】把 YOLO 置信度降到 0.3，让框切得宽松一些（OCR 需要完整的车牌含边缘）
3. 临时关闭"跳过无效车牌"看看原始识别结果是什么

### Q4: 视频里同一辆车被记录了几十次

**这正是"结果去重"功能要解决的问题**：
1. 【⚙ 参数】确保"结果去重"已开启
2. 调整"去重时间窗"：默认 5 秒，长视频场景可设 30-60 秒
3. 清空结果再重新检测，观察表格记录是否减少

### Q5: 有些类型（双层黄牌 / 新能源 / 使馆等）根本检测不出

**原因**：你的 YOLO `best.pt` 只用普通蓝牌训练过，不认识其他类型的外观。

**解决**：用多类型数据集重训 YOLO。推荐：
- [CCPD2020](https://github.com/detectRecog/CCPD) — 20 万张 CCPD 格式中国车牌
- [CRPD](https://github.com/yxgong0/CRPD) — 涵盖新能源、双层等特殊车牌

### Q6: Windows 摄像头打不开

- 关闭其他占用摄像头的程序（微信视频、QQ、Teams 等）
- 菜单【输入源 → 打开摄像头】的索引框里试 1 或 2（外接摄像头通常是 1）
- 代码已经优先使用 `CAP_DSHOW` 后端，Windows 兼容性最好

### Q7: 启动后界面字体是乱码或方块

检查 `config.py` 中 `CHINESE_FONT_PATH` 是否指向一个真实存在的 `.ttf` 文件。如果没有该字体，去下载 `DroidSansFallback.ttf`，或改用系统已有字体：

```python
CHINESE_FONT_PATH = r"C:\Windows\Fonts\msyh.ttc"  # 微软雅黑
```

### Q8: 中文路径的图片读不了

代码已用 `np.fromfile + cv2.imdecode` 和 `cv2.imencode + tofile` 绕过 OpenCV 不支持中文路径的问题。如果你改过代码，注意不要用 `cv2.imread(含中文路径)`。

### Q9: 视频处理卡顿，CPU 满载

【⚙ 参数】把"视频跳帧"从 2 调到 5 或 10，检测速度成倍提升（代价是可能漏掉短暂出现的车牌）。

### Q10: 严格模式后很多正常车牌被过滤掉

"必须含汉字+字母+数字"是严格模式，只保留完全符合"省份汉字+字母+数字"格式的车牌（京A12345 这种）。如果你的场景包含军牌（无汉字）、多国牌（纯英文数字）等，应该**关闭**这个开关。

---

## 🛠 开发者说明

### 关键技术点

| 技术 | 实现位置 | 说明 |
|---|---|---|
| GPU 预热 | `detector.init_gpu()` | 用空白图跑一次 YOLO + OCR，避免首帧延迟甚至崩溃 |
| 暂停/继续 | `workers.DetectionWorker` | `threading.Event`：暂停时 worker 阻塞在 `wait()` |
| 双层车牌 OCR | `detector._recognize_double_layer` | 切分上下两半（上 42% / 下 58%）分别 OCR 再拼接 |
| 深色车牌优化 | `detector._is_dark_plate` | 亮度 < 100 的图额外尝试反色 OCR，取最像车牌号的 |
| 中文路径兼容 | `workers._imread_unicode`、`result_manager._imwrite_unicode` | `np.fromfile` + `imdecode/imencode` 绕过 cv2 |
| 版本双兼容 | `detector.init_gpu()` | 根据 `paddleocr.__version__` 自动走 v2 或 v3 分支 |
| 结果去重 | `result_manager.add()` | 基于 `{号码: 最后出现时间}` 的时间窗去重 |

### 扩展：添加新的车牌类型

修改 `detector.py` 的 `classify_plate_type()` 添加一条判断即可：

```python
if '临' in t:
    return '临时车牌'
```

### 扩展：添加新的过滤规则

在 `detector.is_valid_plate_number()` 里加新分支，在 `main.py` 的 `SettingsDialog` 里加对应控件，在 `config.py` 的 `DEFAULT_SETTINGS` 加默认值即可。参数链路如下：

```
config.DEFAULT_SETTINGS
  → detector.settings (update_settings)
  → result_mgr.settings (update_settings)
  → SettingsDialog 读/写
```

### 调试小贴士

- `detector.OCR_DEBUG = True` 会把每次 OCR 的原始返回打印到终端
- 菜单【模型 → 🔬 诊断 OCR】一键自检整个流水线
- 日志面板的所有记录可以复制出来用于问题反馈

---

## 📄 License

本项目仅供学习与研究使用。YOLO 模型权重、PaddleOCR 模型的使用许可请参考各自原项目。

- YOLO: [Ultralytics License](https://github.com/ultralytics/ultralytics)
- PaddleOCR: [Apache License 2.0](https://github.com/PaddlePaddle/PaddleOCR)
