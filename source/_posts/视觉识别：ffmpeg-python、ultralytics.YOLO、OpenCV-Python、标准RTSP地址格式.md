---
title: 视觉识别：ffmpeg-python、ultralytics.YOLO、OpenCV-Python、标准RTSP地址格式
date: 2026-01-05 08:15:00
tags:
  - "计算机视觉"
  - "YOLO"
  - "OpenCV"
  - "RTSP"
  - "ffmpeg"
  - "ultralytics"
categories:
  - "计算机视觉"
---

﻿@[TOC](视觉识别：ffmpeg-python、ultralytics.YOLO、OpenCV-Python、标准RTSP地址格式)
## ffmpeg-python
`ffmpeg-python` 是一个用于操作 FFmpeg 的 Python 库，它通过 Python 对象和链式调用封装了 FFmpeg 的命令行参数。

---
### 核心概念
1. **`input()`**：创建输入流
   ```python
   input_stream = ffmpeg.input('input.mp4', ss=10, t=5)  # 从第10秒开始，读取5秒
   ```
   - 关键参数：
     - `filename`：输入文件路径
     - `ss`：起始时间（秒或 `HH:MM:SS`）
     - `t`：持续时间
     - `f`：强制输入格式（如 `f='rawvideo'`）

2. **`output()`**：生成输出流
   ```python
   output_stream = ffmpeg.output(input_stream, 'output.mp4', vcodec='libx264', crf=23)
   ```
   - 关键参数：
     - `filename`：输出路径
     - `vcodec`/`acodec`：视频/音频编码器（`'libx264'`, `'aac'`, `'copy'`）
     - `crf`：视频质量（0-51，值越低质量越高）
     - `b:v`/`b:a`：视频/音频比特率（如 `b:v='1M'`）
     - `r`：帧率（如 `r=30`）
     - `s`：分辨率（如 `s='1280x720'`）
     - `preset`：编码速度（`'ultrafast'`, `'slow'`）

3. **`run()`**：执行命令
   ```python
   ffmpeg.run(output_stream, overwrite_output=True)  # 覆盖已存在文件
   ```

---

### 常用过滤器（Filters）
通过 `.filter()` 方法调用：
```python
filtered = input_stream.filter('filter_name', **params)
```

1. **视频缩放**
   ```python
   scaled = input_stream.filter('scale', width=640, height=-1)  # 宽度640，高度按比例
   ```

2. **裁剪**
   ```python
   cropped = input_stream.filter('crop', w=100, h=100, x=20, y=20)
   ```

3. **旋转**
   ```python
   rotated = input_stream.filter('rotate', angle=45*math.pi/180)  # 45度
   ```

4. **叠加水印**
   ```python
   watermarked = ffmpeg.overlay(
       input_stream, 
       ffmpeg.input('watermark.png').filter('scale', 50, 50),
       x=10, y=10
   )
   ```

5. **音频降噪**
   ```python
   denoised = input_stream.filter('afftdn', nf=-20)  # 降噪强度
   ```

---

### 高级操作
1. **合并多个输入**
   ```python
   video = ffmpeg.input('video.mp4')
   audio = ffmpeg.input('audio.mp3')
   output = ffmpeg.output(video, audio, 'merged.mp4', vcodec='copy', acodec='aac')
   ```

2. **提取音轨**
   ```python
   ffmpeg.input('video.mp4').output('audio_only.mp3', acodec='libmp3lame').run()
   ```

3. **生成 GIF**
   ```python
   (
       ffmpeg.input('video.mp4', t=3)
       .filter('fps', fps=10)
       .output('output.gif', loop=0)
       .run()
   )
   ```

4. **硬件加速（NVIDIA）**
   ```python
   (
       ffmpeg.input('input.mp4')
       .output('output.mp4', 
               vcodec='h264_nvenc',   # NVIDIA 编码器
               preset='p1', 
               rc='constqp', 
               qp=21)
       .run()
   )
   ```

---

### 视频截帧转换图片示例

```python
import ffmpeg
import os


def video2img(dir_video, dir_imgs, start_time=60, fps=1, quality=85):
    """从指定时间开始提取关键帧并压缩
    Args:
        start_time: 开始时间（秒）
        fps: 每秒抽取帧数
        quality: JPEG压缩质量
    """
    (
        ffmpeg.input(dir_video, ss=start_time)  # 在输入级别跳转
        .trim(start=0)  # 从跳转后的0秒开始（即start_time位置）
        .filter('fps', fps=fps)  # 控制抽帧率
        .output(dir_imgs,
                q=quality,  # JPEG压缩
                vsync=0)    # 防止帧重复问题
        .run(quiet=True)  # 静默模式
    )


def video2bmp(dir_video, dir_imgs, start_time=60, fps=1):
    """从指定时间开始提取关键帧并输出为BMP
    Args:
        start_time: 开始时间（秒）
        fps: 每秒抽取帧数
    """
    (
        ffmpeg.input(dir_video, ss=start_time)  # 在输入级别跳转
        .trim(start=0)  # 从跳转后的0秒开始（即start_time位置）
        .filter('fps', fps=fps)  # 控制抽帧率
        .output(dir_imgs,
                pix_fmt='rgb24',  # BMP需要RGB格式
                vcodec='bmp',     # 指定输出为BMP格式
                vsync=0)          # 防止帧重复问题
        .run(quiet=True)  # 静默模式
    )

input_dir = './video/'
output_dir = './picture/'
os.makedirs(output_dir, exist_ok=True)

# 配置参数（按需调整）
START_TIME = 1200  # 从第60秒(1分钟)开始
EXTRACT_FPS = 1  # 每秒抽帧数
JPEG_QUALITY = 85  # 压缩质量

for file_name in os.listdir(input_dir):
    if not file_name.lower().endswith(('.mp4', '.avi', '.mov', '.mkv', '.flv')):
        continue

    # 构建路径
    video_path = os.path.join(input_dir, file_name)
    output_subdir = os.path.join(output_dir, os.path.splitext(file_name)[0])
    os.makedirs(output_subdir, exist_ok=True)

    # 输出路径格式 - 改为.bmp后缀
    # output_pattern = os.path.join(output_subdir, 'frame_%04d.bmp')
    output_pattern = os.path.join(output_subdir, 'frame_%04d.jpg')

    try:
        # 从指定时间开始提取
        # video2bmp(
        #     video_path,
        #     output_pattern,
        #     start_time=START_TIME,
        #     fps=EXTRACT_FPS
        # )
        video2img(
            video_path,
            output_pattern,
            start_time=START_TIME,
            fps=EXTRACT_FPS,
            quality=JPEG_QUALITY
        )
        print(f"完成提取: {file_name} (从 {START_TIME} 秒开始)")
    except Exception as e:
        print(f"处理失败: {file_name} - {str(e)}")
```

### 参考
- **官方文档**：[ffmpeg-python GitHub](https://github.com/kkroening/ffmpeg-python)
---
## ultralytics.YOLO（You Only Look Once）
![在这里插入图片描述](/img/de4ebf025422424098426a99e8ed6171.png)

```python
.
├── __init__.py
├── assets: 测试图片
├── cfg: 配置文件，包括数据集、模型、目标跟踪等
├── data: 数据集处理代码，
├── engine: 核心引擎代码，包括YOLO模型结构代码、预测引擎代码、训练器代码
├── hub: Ultralytics Hub登录、连接等相关代码
├── models: 模型定义代码，包含YOLO, SAM, RTDetr，nas等
├── nn: 神经网络模块定义代码，基础组件所在处
├── solutions: 下游任务解决方案代码，如目标跟踪、区域计数、速度估计等
├── trackers: 目标跟踪模块代码，包括具体跟踪算法实现，如bot sort、byte tracker等
└── utils: 工具函数
```

**核心思想：端到端的统一检测**

 - **摒弃传统流程**： 不同于早期的R-CNN系列（R-CNN, Fast R-CNN, Faster R-CNN）等两阶段方法（先提取候选区域，再对每个区域分类和回归），YOLO是单阶段（One-Stage） 方法的代表。
 - **“只看一眼”**： 将整个输入图像一次性输入到一个单一的卷积神经网络（CNN）。
 - **统一预测**： 网络在单次前向传播中，直接预测图像中所有目标的位置（边界框）和类别。它将预测问题建模为一个端到端的回归任务。
### 1. 模型加载
[https://docs.ultralytics.com/zh/models/](https://docs.ultralytics.com/zh/models/)
```python
from ultralytics import YOLO

# 加载预训练模型（支持 .pt, .yaml, 或官方模型名）
model = YOLO("yolov8n.pt")  # 加载预训练权重
# 或
model = YOLO("yolov8n.yaml")  # 从配置文件构建新模型
```

---

### 2. 训练模型 (`train()`)
[https://docs.ultralytics.com/zh/modes/train/#idle-gpu-training](https://docs.ultralytics.com/zh/modes/train/#idle-gpu-training)
```python
model.train(
    data='dataset.yaml',   # 数据集配置文件路径
    epochs=100,            # 训练轮次
    batch=16,              # 批次大小
    imgsz=640,             # 输入图像尺寸
    device='cuda',         # 训练设备
    workers=8,             # 数据加载线程数
    optimizer='SGD',       # 优化器类型
    lr0=0.01,              # 初始学习率
    weight_decay=0.0005,   # 权重衰减
    augment=True,          # 数据增强开关
    # 增强参数：
    hsv_h=0.015,           # 色相增强幅度
    fliplr=0.5,            # 左右翻转概率
    mosaic=1.0,            # Mosaic增强概率
    mixup=0.1,             # Mixup增强概率
    # 输出控制：
    project='runs/train',  # 结果保存目录
    name='exp1',           # 实验名称
    save_period=10,        # 每N轮保存一次检查点
    resume=False,          # 恢复训练
)
```
#### 标准YAML格式示例
在YOLO（尤其是YOLOv5/YOLOv8）中，用于训练的数据集配置文件（如 `basket_random_split.yaml`）需要遵循特定格式。
```yaml
# basket_random_split.yaml
path: ./datasets/basket  # 数据集根目录
train: images/train      # 训练集路径（相对于path）
val: images/val          # 验证集路径（相对于path）
test: images/test        # 测试集路径（可选）

# 类别数量
nc: 3

# 类别名称列表（按索引顺序）
names: 
  0: basketball
  1: hoop
  2: player
```

 -  **`names`** (必须)  
	   - 类别名称列表（字符串数组或字典）
	   - 索引必须从0开始连续编号
 	  - 两种写法：
     ```yaml
     # 数组写法
     names: ['basketball', 'hoop', 'player']
     
     # 字典写法
     names: 
       0: basketball
       1: hoop
       2: player
     ```

> **重要提示**：YOLOv5/YOLOv8会自动根据`images`路径查找对应的`labels`目录，确保目录结构匹配。如果测试集未标注，可省略`test`字段。
---
### 3. 预测 (`predict()`)
[https://docs.ultralytics.com/zh/modes/predict/#inference-arguments](https://docs.ultralytics.com/zh/modes/predict/#inference-arguments)
```python
model.predict(
    source='path/to/data', # 数据源 (图像/视频/目录/URL)
    conf=0.25,             # 置信度阈值
    iou=0.45,              # NMS IoU阈值
    # 输出控制：
    show=True,             # 实时显示结果
    save=True,             # 保存带检测框的图像
    save_txt=False,        # 保存YOLO格式标签
    save_conf=True,        # 在标签中包含置信度
    save_crop=False,       # 保存裁剪的目标
    # 目标过滤：
    classes=[0, 2],        # 只检测特定类别
    agnostic_nms=False,    # 类别无关的NMS
    # 性能优化：
    stream=False,          # 流模式 (减少内存)
    max_det=300,           # 每图最大检测数
    half=True,             # 使用FP16半精度
    # 可视化：
    line_width=3,          # 边界框线宽
    visualize=False,       # 生成特征可视化
)
```

---

### 4. 验证模型 (`val()`)
[https://docs.ultralytics.com/zh/modes/val/#arguments-for-yolo-model-validation](https://docs.ultralytics.com/zh/modes/val/#arguments-for-yolo-model-validation)
```python
model.val(
    data='dataset.yaml',   # 数据集配置文件
    split='val',           # 使用哪个数据划分 (val/test)
    batch=32,              # 批次大小
    imgsz=640,             # 输入图像尺寸
    conf=0.001,            # 置信度阈值
    iou=0.6,               # NMS IoU阈值
    device='cuda',         # 推理设备
    # 输出控制：
    save_json=True,        # 保存COCO格式结果
    save_hybrid=True,      # 保存混合标签结果
    plots=True,            # 生成评估图表
    half=True,             # 使用FP16半精度
    # 特殊选项：
    rect=False,            # 是否使用矩形推理
    dnn=False,             # 是否使用ONNX DNN
)
```
#### metrics
model.val() 方法返回的 metrics 对象包含了丰富的模型性能指标
##### 核心指标 (metrics.box)

| 属性                   | 类型        | 说明                                   |
| :--------------------- | :---------- | :------------------------------------- |
| **`map`**              | float       | mAP@0.5:0.95 (IoU 0.5-0.95 的平均精度) |
| **`map50`**            | float       | mAP@0.5 (IoU=0.5 的平均精度)           |
| **`map75`**            | float       | mAP@0.75 (IoU=0.75 的平均精度)         |
| **`maps`**             | list[float] | 每个类别的 mAP@0.5:0.95                |
| **`ap50`**             | list[float] | 每个类别的 AP@0.5                      |
| **`ap`**               | list[float] | 每个类别的 AP@0.5:0.95                 |
| **`precision`**        | list[float] | 每个类别的精确率 (Precision)           |
| **`recall`**           | list[float] | 每个类别的召回率 (Recall)              |
| **`f1`**               | list[float] | 每个类别的 F1 分数                     |
| **`ap_class_index`**   | list[int]   | 对应类别的索引                         |
| **`confusion_matrix`** | np.ndarray  | 混淆矩阵                               |

------
##### 其他重要属性

| 属性               | 类型      | 说明                          |
| :----------------- | :-------- | :---------------------------- |
| **`speed`**        | dict      | 处理速度指标                  |
| - `preprocess`     | float     | 预处理时间 (ms/图像)          |
| - `inference`      | float     | 推理时间 (ms/图像)            |
| - `loss`           | float     | 损失计算时间 (ms/图像)        |
| - `postprocess`    | float     | 后处理时间 (ms/图像)          |
| **`results_dict`** | dict      | 所有结果的字典形式            |
| **`class_names`**  | list[str] | 类别名称列表                  |
| **`nt_per_class`** | list[int] | 每个类别的真实目标数量        |
| **`mp`**           | float     | 平均精确率 (所有类别的均值)   |
| **`mr`**           | float     | 平均召回率 (所有类别的均值)   |
| **`fi`**           | float     | 平均 F1 分数 (所有类别的均值) |

------
##### 完整指标获取示例


```python
metrics = model.val(
    data='dataset.yaml',
    split='val',
    conf=0.25,
    iou=0.6,
    plots=True
)

# 1. 核心指标
print(f"综合评估指标:")
print(f"  mAP@0.5: {metrics.box.map50:.4f}")
print(f"  mAP@0.5:0.95: {metrics.box.map:.4f}")
print(f"  平均精确率: {metrics.box.mp:.4f}")
print(f"  平均召回率: {metrics.box.mr:.4f}")
print(f"  平均F1分数: {metrics.box.fi:.4f}")

# 2. 类别详细指标
class_names = metrics.class_names
print("\n各类别详细指标:")
for i, cls_idx in enumerate(metrics.box.ap_class_index):
    print(f"  {class_names[cls_idx]} (ID:{cls_idx}):")
    print(f"    AP@0.5: {metrics.box.ap50[i]:.4f}")
    print(f"    AP@0.5:0.95: {metrics.box.ap[i]:.4f}")
    print(f"    精确率: {metrics.box.precision[i]:.4f}")
    print(f"    召回率: {metrics.box.recall[i]:.4f}")
    print(f"    F1分数: {metrics.box.f1[i]:.4f}")
    print(f"    真实目标数: {metrics.nt_per_class[i]}")

# 3. 速度指标
print("\n处理速度:")
print(f"  预处理: {metrics.speed['preprocess']:.2f} ms/图像")
print(f"  推理: {metrics.speed['inference']:.2f} ms/图像")
print(f"  后处理: {metrics.speed['postprocess']:.2f} ms/图像")

# 4. 其他指标
print(f"\n混淆矩阵形状: {metrics.confusion_matrix.shape}")
print(f"结果字典键: {list(metrics.results_dict.keys())}")
```


------

##### results_dict 中的完整键值

当访问 `metrics.results_dict` 时，会得到包含所有指标的字典：

```python
{
    'metrics/precision(B)': 平均精确率,
    'metrics/recall(B)': 平均召回率,
    'metrics/mAP50(B)': mAP@0.5,
    'metrics/mAP50-95(B)': mAP@0.5:0.95,
    'fitness': 综合适应度分数,
    'val/box_loss': 验证集边界框损失,
    'val/cls_loss': 验证集分类损失,
    'val/dfl_loss': 验证集分布焦点损失,
    'x/lr0': 当前学习率,
    'x/lr1': 当前学习率(第二组),
    'x/lr2': 当前学习率(第三组)
}
```

------

##### 注意事项

1. **指标前缀说明**：
   - `(B)` 表示边界框相关指标
   - `(M)` 表示掩码相关指标（实例分割任务）
   - `(P)` 表示姿态相关指标（姿态估计任务）
2. **指标计算依据**：
   - 所有指标基于设定的置信度阈值(`conf`)和IoU阈值(`iou`)
   - 默认使用0.001置信度阈值来最大化召回率
3. **可视化输出**：
   当设置 `plots=True` 时，会额外生成：
   - PR曲线图
   - 混淆矩阵图
   - 各类别误差分析图
   - 预测结果示例图


---
### 5. 模型导出 (`export()`)
```python
model.export(
    format="onnx",            # 格式：onnx, engine, coreml, tflite...
    imgsz=(640, 480),         # 输入尺寸（高, 宽）
    dynamic=False,            # ONNX/TensorRT：动态维度
    simplify=True,            # ONNX：简化模型
    opset=12,                 # ONNX：算子集版本
    half=True,                # FP16量化
    device="cpu",             # 导出设备
)
```

---

### 常用参数说明表
| 参数          | 适用方法       | 说明                                                                 |
|---------------|---------------|----------------------------------------------------------------------|
| `data`        | `train`, `val` | 数据集YAML配置文件路径                                               |
| `epochs`      | `train`       | 训练总轮次                                                           |
| `imgsz`       | 所有方法       | 输入图像尺寸（整数或元组）                                           |
| `conf`        | `predict`, `val` | 置信度阈值（0-1）                                                   |
| `iou`         | `predict`, `val` | NMS的IoU阈值（0-1）                                                 |
| `device`      | 所有方法       | 运行设备（`cpu`, `cuda`, `mps`）                                     |
| `batch`       | `train`, `val` | 批次大小（`-1`表示自动批大小）                                       |
| `project`     | `train`       | 结果保存的根目录（如 `runs/train`）                                  |
| `name`        | `train`       | 实验名称（在`project`下生成子目录）                                  |
| `classes`     | `predict`     | 指定检测的类别ID列表                                                 |
| `save_txt`    | `predict`     | 保存结果为YOLO格式的txt标签                                          |
| `save_json`   | `val`         | 保存COCO格式的JSON结果                                               |
| `format`      | `export`      | 导出格式（`onnx`, `engine`, `tflite`, `coreml`等）                   |

---
#### 训练验证输出中各参数的详细解释

```python
      Epoch    GPU_mem   box_loss   cls_loss   dfl_loss  Instances       Size
       1/30         0G      1.575      3.235      1.696         20        640: 100%|██████████| 20/20 [07:29<00:00, 22.47s/it]
                 Class     Images  Instances      Box(P          R      mAP50  mAP50-95): 100%|██████████| 3/3 [00:30<00:00, 10.14s/it]
                   all         78         78      0.817      0.938      0.977      0.715
```
**训练迭代参数（每轮训练）**
| 参数 | 说明 | 示例值 | 解读 |
|------|------|--------|------|
| **Epoch** | 当前训练轮次/总轮次 | 1/30 | 第1轮训练（共30轮） |
| **GPU_mem** | GPU显存占用 | 0G | 显存使用较低（单位：GB） |
| **box_loss** | 边界框回归损失 | 1.575 | 预测框位置误差，值越小定位越准 |
| **cls_loss** | 分类损失 | 3.235 | 类别预测误差，值越小分类越准 |
| **dfl_loss** | 分布聚焦损失 | 1.696 | 边界框分布离散化误差（YOLOv8特有） |
| **Instances** | 当前批次目标数 | 20 | 本批含20个待检测目标 |
| **Size** | 输入图像尺寸 | 640 | 图像缩放至640×640像素 |
| **进度** | 批次进度 | 20/20 | 完成20个训练批次 |
| **时间** | 训练耗时 | 07:29<00:00 | 本轮耗时7分29秒 |
| **速度** | 批次处理速度 | 22.47s/it | 每批次平均耗时22.47秒 |

---

**验证阶段参数（每轮训练后评估）**
| 参数 | 说明 | 示例值 | 解读 |
|------|------|--------|------|
| **Class** | 评估类别 | all | 所有类别的综合指标 |
| **Images** | 验证图片数 | 78 | 使用78张图片评估模型 |
| **Instances** | 目标实例数 | 78 | 验证集含78个待检测目标 |
| **P** | 边界框精确率 | 0.817 | 81.7%的预测框正确（查准率） |
| **R** | 边界框召回率 | 0.938 | 93.8%的目标被成功检出（查全率） |
| **mAP50** | IoU=0.5的mAP | 0.977 | 97.7%的高精度检测（核心指标） |
| **mAP50-95** | IoU[0.5:0.95]的mAP | 0.715 | 严格IoU阈值下的平均精度（鲁棒性指标） |
| **进度** | 验证批次进度 | 3/3 | 完成3个验证批次 |
| **时间** | 验证耗时 | 00:30<00:00 | 总耗时30秒 |
| **速度** | 批次处理速度 | 10.14s/it | 每批次平均耗时10.14秒 |

---
### train/val/predict关键区别总结

| 特性         | `train()`      | `val()`        | `predict()`   |
| :----------- | :------------- | :------------- | :------------ |
| **主要目的** | 模型训练       | 性能评估       | 新数据推理    |
| **数据源**   | 数据集配置文件 | 数据集配置文件 | 任意图像/视频 |
| **增强处理** | ✓ 数据增强     | ✗ 无增强       | ✗ 无增强      |
| **输出类型** | 训练日志/模型  | 评估指标       | 检测结果图像  |
| **关键指标** | 训练损失       | mAP/Precision  | 检测框/置信度 |
| **设备优化** | 训练优化       | 评估优化       | 推理优化      |
| **参数特点** | 学习率/优化器  | 评估阈值       | 可视化选项    |
| **典型使用** | 训练前         | 训练后评估     | 部署推理      |

------
### mAP 详解

#### 1. 核心概念

| 术语          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| **IoU**       | **交并比（Intersection over Union）<br>预测框与真实框的重叠程度** |
| **TP**        | 真正例（True Positive）<br>IoU ≥ 阈值（如0.5）的正确检测     |
| **FP**        | 假正例（False Positive）<br>IoU < 阈值或重复检测             |
| **FN**        | 假负例（False Negative）<br>未检测到的真实目标               |
| **Precision** | TP / (TP + FP)<br>检测结果的准确性                           |
| **Recall**    | TP / (TP + FN)<br>目标检测的完整性                           |

$$IoU=\frac{预测框与真实框的交集面积}{并集面积}$$
#### 2. 计算流程

1. **生成PR曲线**：
   - 对每个类别，在不同置信度阈值下计算Precision和Recall
   - 绘制Precision-Recall曲线

2. **计算AP（Average Precision）**：
   - 计算PR曲线下的面积
   - 公式：$AP = \int_0^1 p(r) dr$
   - 实际计算：对Recall进行插值后取平均精度

3. **计算mAP**：
   - 对所有类别的AP取平均值
   - 公式：$mAP = \frac{1}{N}\sum_{i=1}^{N} AP_i$

#### 3. 常见变体

| 类型             | 说明                             | 应用场景           |
| ---------------- | -------------------------------- | ------------------ |
| **mAP@0.5**      | IoU阈值固定为0.5                 | 基础评估标准       |
| **mAP@0.5:0.95** | IoU从0.5到0.95（步长0.05）取平均 | 严格评估定位精度   |
| **mAP@0.75**     | IoU阈值固定为0.75                | 高精度定位要求场景 |

---

#### mAP 在目标检测中的意义

1. **综合性**：同时考虑精确率和召回率
2. **鲁棒性**：不受置信度阈值选择的影响
3. **类别平衡**：平等对待所有类别
4. **定位敏感**：反映边界框的准确度（通过IoU）

---
#### mAP 解读指南

| mAP值范围 | 模型性能           |
| --------- | ------------------ |
| 0.9+      | 极好（接近完美）   |
| 0.7-0.9   | 优秀（工业应用级） |
| 0.5-0.7   | 可用（需优化）     |
| 0.3-0.5   | 较差（需大幅改进） |
| <0.3      | 基本无效           |

---

#### 提升mAP的策略

1. **数据层面**：
   - 增加困难样本
   - 平衡类别分布
   - 优化标注质量

2. **模型层面**：
   - 使用更大模型（如YOLOv8x）
   - 增加输入分辨率（`imgsz`）
   - 调整锚框尺寸

3. **训练技巧**：
   - 延长训练时间（增加`epochs`）
   - 调整学习率策略
   - 增强数据多样性（`augment=True`）

4. **后处理优化**：
   - 调整NMS参数（`iou`）
   - 优化置信度阈值（`conf`）

mAP 是目标检测领域的"黄金标准"，全面反映了模型在定位准确性和分类正确性上的综合能力。在工业应用中，mAP@0.5 > 0.7 通常是可部署的最低标准。

### runs/detect日志讲解
  - weights 是训练好的模型权重，保留了best.pt,last.pt
  - args.yaml 是训练过程中的参数记录。
  - results.csv 是训练过程中的记录。
  - results.png 是训练过程中的记录的可视化。
  - confusion_matrix.png ，confusion_matrix_normalized.png 是训练过程中的混淆矩阵的可视化和可视化归一化。
  - labels.jpg，labels_correlogram.jpg 是训练过程中的标签和标签的correlogram。
  - F1_curve.png，PR_curve.png，P_curve.png，R_curve.png 是训练过程中的F1曲线、PR曲线、P曲线、R曲线。
  - train_batch0.jpg，train_batch1.jpg，train_batch2.jpg 是训练过程中的训练图片，采用了mosaic的数据增强方式。
  - train_batch23250.jpg，train_batch23251.jpg，train_batch23252.jpg 是训练过程中的训练图片，不再采用mosaic的数据增强方式。
  - val_batch0_labels.jpg，val_batch1_labels.jpg ，val_batch2_labels.jpg 是训练过程中的验证图片的标签。
  - val_batch0_labels.jpg，val_batch1_labels.jpg ，val_batch2_labels.jpg 是训练过程中的验证图片的标签。
  - val_batch0_pred.jpg，val_batch1_pred.jpg ，val_batch2_pred.jpg 是训练过程中的验证图片的预测值。
### 参考

 - 官方文档参考：[Ultralytics YOLO Docs](https://docs.ultralytics.com)
 - [YOLO系列模型之如何阅读ultralytics源码？](https://wvet00aj34c.feishu.cn/docx/K4d9d9B5KoaSPjxwOjXceBwKnih)
 - 项目基础配置说明：[https://docs.ultralytics.com/quickstart/#ultralytics-settings](https://docs.ultralytics.com/quickstart/#ultralytics-settings)
 - 下游任务解决方案：[https://docs.ultralytics.com/solutions/](https://docs.ultralytics.com/solutions/)
 - 研发过程相关指导：[https://docs.ultralytics.com/guides/](https://docs.ultralytics.com/guides/)
 - [LabelImg](https://github.com/HumanSignal/labelImg)、[X-AnyLabeling](https://github.com/CVHub520/X-AnyLabeling/blob/main/docs/zh_cn/user_guide.md)

---
## OpenCV-Python（cv2）
### 1. 图像读取与保存
#### `cv2.imread(filename, flags)`
- **功能**: 读取图像文件。
- **参数**:
  - `filename`: 图像路径（支持 JPG、PNG、TIFF 等格式）。
  - `flags`: 读取模式（可选）:
    - `cv2.IMREAD_COLOR` (默认): 加载 3 通道 BGR 彩色图像。
    - `cv2.IMREAD_GRAYSCALE`: 加载灰度图像。
    - `cv2.IMREAD_UNCHANGED`: 保留原始通道（如 PNG 的透明度）。
- **返回值**: `numpy.ndarray` 格式的图像数据。

#### `cv2.imwrite(filename, img, params)`
- **功能**: 保存图像。
- **参数**:
  - `filename`: 保存路径（扩展名决定格式）。
  - `img`: 要保存的图像数据。
  - `params` (可选): 编码参数（如 JPEG 质量）:
    - `[cv2.IMWRITE_JPEG_QUALITY, 95]`: JPEG 质量（0-100）。
    - `[cv2.IMWRITE_PNG_COMPRESSION, 9]`: PNG 压缩级别（0-9）。

#### `cv2.imshow()`
- **功能**: 在窗口中显示图像。
- **用法**: cv2.imshow("窗口名", image)
---
### 2. 图像处理
#### `cv2.cvtColor(src, code)`
- **功能**: 颜色空间转换。
- **参数**:
  - `src`: 输入图像。
  - `code`: 转换类型:
    - `cv2.COLOR_BGR2GRAY`: BGR → 灰度。
    - `cv2.COLOR_BGR2HSV`: BGR → HSV。
    - `cv2.COLOR_BGR2RGB`: BGR → RGB。

#### `cv2.resize(src, dsize, fx, fy, interpolation)`
- **功能**: 调整图像尺寸。
- **参数**:
  - `dsize`: 目标尺寸 `(width, height)`。
  - `fx`, `fy`: 沿 x/y 轴的缩放因子。
  - `interpolation`: 插值方法（默认 `cv2.INTER_LINEAR`）:
    - `cv2.INTER_NEAREST`: 最近邻插值（快）。
    - `cv2.INTER_CUBIC`: 双三次插值（慢但质量高）。
##### 为什么需要插值？

图像缩放时，**输入与输出像素的位置并非一一对应**：

- **放大图像**：输出像素可能位于输入像素之间
- **缩小图像**：多个输入像素映射到同一输出位置
  插值通过**数学估算**解决这些位置的像素值问题，避免锯齿、模糊等失真。
#### `cv2.GaussianBlur(src, ksize, sigmaX)`
- **功能**: 高斯模糊（降噪）。
- **参数**:
  - `ksize`: 高斯核大小 `(width, height)`（必须为奇数）。
  - `sigmaX`: X 方向标准差（若为 0，则根据 ksize 自动计算）。

---

### 3. 阈值与二值化
#### `cv2.threshold(src, thresh, maxval, type)`
- **功能**: 图像阈值处理。
- **参数**:
  - `thresh`: 阈值（0-255）。
  - `maxval`: 超过阈值时赋予的值。
  - `type`: 阈值类型:
    - `cv2.THRESH_BINARY`: `dst = (src > thresh) ? maxval : 0`。
    - `cv2.THRESH_OTSU`: 自动计算阈值（需与 `cv2.THRESH_BINARY` 组合使用）。
- **返回值**: `retval`（实际使用的阈值）, `dst`（二值化图像）。

---

### 4. 特征检测
#### `cv2.Canny(image, threshold1, threshold2, apertureSize, L2gradient)`
- **功能**: Canny 边缘检测。
- **参数**:
  - `threshold1`: 低阈值（弱边缘过滤）。
  - `threshold2`: 高阈值（强边缘保留）。
  - `apertureSize`: Sobel 算子大小（默认 3）。
  - `L2gradient`: 是否使用更精确的 L2 范数（默认 False，用 L1）。

#### `cv2.HoughLinesP(image, rho, theta, threshold, minLineLength, maxLineGap)`
- **功能**: 概率霍夫变换检测直线。
- **参数**:
  - `rho`: 距离分辨率（像素）。
  - `theta`: 角度分辨率（弧度）。
  - `threshold`: 投票阈值（低于此值忽略）。
  - `minLineLength`: 线段最小长度。
  - `maxLineGap`: 线段最大间断距离。

---

### 5. 视频处理
#### `cv2.VideoCapture(index)`
- **功能**: 打开摄像头或视频文件。
- **参数**:
  - `index`: 摄像头 ID（0 为默认摄像头）或视频文件路径。
- **常用方法**:
  - `cap.read()`: 读取帧（返回 `ret, frame`）。
  - `cap.set(propId, value)`: 设置属性（如 `cv2.CAP_PROP_FPS`, `cv2.CAP_PROP_FRAME_WIDTH`）。
  - `cap.release()`: 释放资源。

#### `cv2.VideoWriter(filename, fourcc, fps, frameSize)`
- **功能**: 保存视频。
- **参数**:
  - `fourcc`: 编码器（如 `cv2.VideoWriter_fourcc(*'XVID')`）。
  - `fps`: 帧率（如 30）。
  - `frameSize`: 帧尺寸 `(width, height)`。

---

### 6. 绘图函数
#### `cv2.rectangle(img, pt1, pt2, color, thickness)`
- **参数**:
  - `pt1`, `pt2`: 矩形对角点 `(x1,y1)`, `(x2,y2)`。
  - `color`: BGR 元组（如 `(0, 255, 0)` 表示绿色）。
  - `thickness`: 线宽（-1 表示填充）。

#### `cv2.putText(img, text, org, fontFace, fontScale, color, thickness)`
- **参数**:
  - `org`: 文本左下角坐标 `(x, y)`。
  - `fontFace`: 字体（如 `cv2.FONT_HERSHEY_SIMPLEX`）。
  - `fontScale`: 字体缩放因子。

---

### 7. 形态学操作
#### `cv2.erode(src, kernel, iterations)`
- **功能**: 腐蚀（缩小白色区域）。
- **参数**:
  - `kernel`: 结构元素（如 `np.ones((3,3), np.uint8)`）。
  - `iterations`: 执行次数。

#### `cv2.dilate(src, kernel, iterations)`
- **功能**: 膨胀（扩大白色区域）。

#### `cv2.morphologyEx(src, op, kernel)`
- **功能**: 高级形态学操作。
- **参数**:
  - `op`: 操作类型:
    - `cv2.MORPH_OPEN`: 开运算（去噪）。
    - `cv2.MORPH_CLOSE`: 闭运算（填充空洞）。



### 8. 窗口管理

| **参数/函数**              | **说明**           | **常用值/用法**                                              |
| -------------------------- | ------------------ | ------------------------------------------------------------ |
| **`namedWindow()`**        | 创建命名窗口       | `cv2.namedWindow("窗口名", flags)`<br>• `flags=cv2.WINDOW_NORMAL` (可调整大小)<br>• `flags=cv2.WINDOW_AUTOSIZE` (自动适应图像) |
| **`WND_PROP_FULLSCREEN`**  | 窗口全屏属性标识符 | 用于`setWindowProperty()`/`getWindowProperty()`              |
| **`WINDOW_FULLSCREEN`**    | 设置全屏模式的标志 | `cv2.setWindowProperty("win", cv2.WND_PROP_FULLSCREEN, cv2.WINDOW_FULLSCREEN)` |
| **`setWindowProperty()`**  | 设置窗口属性       | `cv2.setWindowProperty("win", 属性, 值)`<br>• 属性：`cv2.WND_PROP_FULLSCREEN`<br>• 值：`cv2.WINDOW_FULLSCREEN` |
| **`getWindowImageRect()`** | 获取窗口坐标和尺寸 | `x, y, w, h = cv2.getWindowImageRect("窗口名")`              |
| **`resizeWindow()`**       | 调整窗口尺寸       | `cv2.resizeWindow("窗口名", 宽, 高)`                         |
| **`destroyWindow()`**      | 关闭单个窗口       | `cv2.destroyWindow("窗口名")`                                |
| **`destroyAllWindows()`**  | 关闭所有窗口       | `cv2.destroyAllWindows()`                                    |


---
## 标准RTSP地址格式

```bash
rtsp://[用户名]:[密码]@[相机IP地址]:[端口]/cam/realmonitor?channel=[通道号]&subtype=[码流类型]
```

### 参数说明

1. **`[用户名]`**  
   - 相机登录用户名（如 `admin`）。

2. **`[密码]`**  
   - 该用户名的密码。

3. **`[相机IP地址]`**  
   - 相机的网络IP（如 `192.168.1.108`）。

4. **`[端口]`**（可选）  
   - RTSP服务端口，**默认为 `554`**（若未修改可省略）。

5. **`[通道号]`**  
   - **`channel=1`**：单路相机固定为 `1`；  
   - NVR下的相机：按实际通道顺序填写（如 `channel=2` 表示第2通道）。

6. **`[码流类型]`**  
   - **`subtype=0`**：主码流（高清，默认值）；  
   - **`subtype=1`**：子码流（低码率，适用于移动端或带宽受限场景）。

---
### 调试工具建议

- 使用 **VLC播放器** 测试：  
  `媒体` → `打开网络串流` → 粘贴RTSP地址 → 播放。
- 工具：**ONVIF Device Manager** 扫描相机，自动生成RTSP地址。


### 参考

 -  [OpenCV Python  Documentation](https://docs.opencv.org/4.x/d6/d00/tutorial_py_root.html)
 - [https://www.runoob.com/opencv/opencv-tutorial.html](https://www.runoob.com/opencv/opencv-tutorial.html)
 - [https://sxwqtaijh4.feishu.cn/docx/WNLJdo0wxoFPuExt6rbcvB8MnPg](https://sxwqtaijh4.feishu.cn/docx/WNLJdo0wxoFPuExt6rbcvB8MnPg)

