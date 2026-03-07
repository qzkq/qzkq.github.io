---
title: PaddleOCR全版本完整指南：从入门到精通，全面解析安装部署与高级应用
date: 2026-01-02 08:00:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](PaddleOCR全版本完整指南：从入门到精通，全面解析安装部署与高级应用)
OCR 的全称是光学字符识别，是一种将图像中的文字（无论是打印体、手写体还是场景文字）自动识别并转换为可编辑、可搜索的文本数据（如TXT、Word、Excel格式）的技术。
## 第一部分：安装与环境配置

### 1. 环境准备

推荐使用**Python 3.11**。你可以使用Anaconda或Miniconda创建一个独立的虚拟环境，这能有效避免不同项目间的包冲突。

```bash
# 创建并激活一个名为paddle_env的虚拟环境
conda create -n paddle_env python=3.11
conda activate paddle_env
```

### 2. 安装PaddlePaddle深度学习框架

这是PaddleOCR运行的基础。你需要根据电脑是否有NVIDIA显卡，选择安装**CPU版本**或**GPU版本**。

- **CPU版本**（通用，适合所有电脑）：

  ```bash
  python -m pip install paddlepaddle==3.2.2 -i https://www.paddlepaddle.org.cn/packages/stable/cpu/
  ```

- **GPU版本**（速度快，需要有NVIDIA显卡和CUDA工具包）：
  安装命令会根据你的CUDA版本有所不同。例如，对于CUDA 12.6，你可以使用：

  ```bash
   python -m pip install paddlepaddle-gpu==3.2.2 -i https://www.paddlepaddle.org.cn/packages/stable/cu126/
  ```

  更详细的版本匹配信息，建议查阅[PaddlePaddle官方安装文档](https://www.paddlepaddle.org.cn/install/quick)。

### 3. 安装PaddleOCR库

安装好框架后，就可以安装PaddleOCR了。

```bash
# 只希望使用基础文字识别功能（返回文字位置坐标和文本内容）
python -m pip install paddleocr
# 希望使用文档解析、文档理解、文档翻译、关键信息抽取等全部功能
# python -m pip install "paddleocr[all]"
```

除了上面演示的 `all` 依赖组以外，PaddleOCR 也支持通过指定其它依赖组，安装部分可选功能。PaddleOCR 提供的所有依赖组如下：

| 依赖组名称   | 对应的功能                                                   |
| :----------- | :----------------------------------------------------------- |
| `doc-parser` | 文档解析，可用于提取文档中的表格、公式、印章、图片等版面元素，包含 PP-StructureV3 等模型方案 |
| `ie`         | 信息抽取，可用于从文档中提取关键信息，如姓名、日期、地址、金额等，包含 PP-ChatOCRv4 等模型方案 |
| `trans`      | 文档翻译，可用于将文档从一种语言翻译为另一种语言，包含 PP-DocTranslation 等模型方案 |
| `all`        | 完整功能                                                     |

更详细的版本匹配信息，建议查阅[安装 - PaddleOCR 文档](https://www.paddleocr.ai/latest/version3.x/installation.html)。

### 4. 验证安装与初步使用

```python
from paddleocr import PPStructureV3

pipeline = PPStructureV3(
    use_doc_orientation_classify=False,
    use_doc_unwarping=False)
output = pipeline.predict(
    input="./pp_structure_v3_demo.png",          
)
for res in output:
    res.print()
    res.save_to_json(save_path="output")
    res.save_to_markdown(save_path="output")
```

如果程序没有报错，并打印出了识别出的文字及其在图片中的位置坐标和置信度，说明安装成功！

```python
{'res': {'input_path': './pp_structure_v3_demo.png', 'page_index': None, 'model_settings': {'use_doc_preprocessor': False, 'use_seal_recognition': True, 'use_table_recognition': True, 'use_formula_recognition': True, 'use_chart_recognition': False, 'use_region_detection': True}, 'layout_det_res': {'input_path': None, 'page_index': None, 'boxes': [{'cls_id': 1, 'label': 'image', 'score': 0.9864752888679504, 'coordinate': [774.821, 201.05177, 1502.1008, 685.7733]}, {'cls_id': 2, 'label': 'text', 'score': 0.9859225749969482, 'coordinate': [769.8655, 776.2446, 1121.5986, 1058.417]}, {'cls_id': 2, 'label': 'text', 'score': 0.9857110381126404, 'coordinate': [1151.98, 1112.5356, 1502.7852, 1346.3569]}, {'cls_id': 10, 'label': 'doc_title', 'score': 0.9376171827316284, 'coordinate': [133.77905, 36.8844, 1379.6667, 123.46869]}, {'cls_id': 2, 'label': 'text', 'score': 0.9020252823829651, 'coordinate': [584.9165, 159.1416, 927.22876, 179.01605]}, {'cls_id': 2, 'label': 'text', 'score': 0.895164430141449, 'coordinate': [1154.3364, 776.74646, 1331.8564, 794.2301]}, {'cls_id': 6, 'label': 'figure_title', 'score': 0.7892374396324158, 'coordinate': [808.9641, 704.2555, 1484.0623, 747.2296]}]}, 'overall_ocr_res': {'input_path': None, 'page_index': None, 'model_settings': {'use_doc_preprocessor': False, 'use_textline_orientation': False}, 'dt_polys': array([[[ 129,   42],
        ...,
        [ 129,  140]],

    ...,

    [[1156, 1330],
        ...,
        [1156, 1351]]], dtype=int16), 'text_det_params': {'limit_side_len': 736, 'limit_type': 'min', 'thresh': 0.3, 'max_side_limit': 4000, 'box_thresh': 0.6, 'unclip_ratio': 1.5}, 'text_type': 'general', 'textline_orientation_angles': array([-1, ..., -1]), 'text_rec_score_thresh': 0.0, 'rec_texts': ['助力双方交往', '搭建友谊桥梁', '本报记者沈小晓', '任', '彦', '黄培昭', '身着中国传统民族服装的厄立特里亚青', '厄立特里亚高等教育与研究院合作建立，开', '年依次登台表演中国民族舞、现代舞、扇子舞', '烂文明。"'], 'rec_scores': array([0.99113536, ..., 0.95110035]), 'rec_polys': array([[[ 129,   42],
        ...,
        [ 129,  140]],

    ...,

    [[1156, 1330],
        ...,
        [1156, 1351]]], dtype=int16), 'rec_boxes': array([[ 129, ...,  140],
    ...,
    [1156, ..., 1351]], dtype=int16)}}}
```

### 5. 常见问题

#### Windows系统用户名为中文时，下载模型会报错，需要用户名为英文

```python
RuntimeError: (NotFound) Cannot open file C:\Users\***\.paddlex\official_models\PP-DocBlockLayout\inference.json, please confirm whether the file is normal.
```
#### 手动下载模型

在一些生产环境或无法自动下载的情况下，您可以手动准备模型：

1.  **获取模型文件**：从[PaddleOCR的官方渠道](https://www.paddleocr.ai/latest/version3.x/module_usage/doc_img_orientation_classification.html)下载所需的推理模型文件。这些文件通常是压缩包。
2.  **解压模型**：将下载的压缩包解压到一个您选定的文件夹中。
3.  **配置路径**：在自定义的YAML配置文件中，将对应模型的 `model_dir` 设置指向这个解压后的文件夹路径。


## 第二部分：PPStructureV3文档解析
PP-StructureV3 能够将文档图像和 PDF 文件高效转换为结构化内容（如 Markdown 格式），并具备版面区域检测、表格识别、公式识别、图表理解以及多栏阅读顺序恢复等强大功能。
### 1. PPStructureV3快速开始

```python
from paddleocr import PPStructureV3

pipeline = PPStructureV3(
    use_doc_orientation_classify=False,
    use_doc_unwarping=False)
output = pipeline.predict(
    input="./pp_structure_v3_demo.png",          
)
for res in output:
    res.print()
    res.save_to_json(save_path="output")
    res.save_to_markdown(save_path="output")
```

### 2. PPStructureV3参数详解

| **参数分类**       | **参数名称**                                               | **功能与默认值**                                             | **什么时候需要调整**                                         |
| :----------------- | :--------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **核心功能开关**   | `use_doc_orientation_classify`                             | **是否启用整图方向分类**。默认值：`False`。<br>自动检测并纠正0/90/180/270度旋转的文档。 | 处理手机拍摄或扫描仪产生的**方向不正确**的文档图片时，设为 `True`。 |
|                    | `use_doc_unwarping`                                        | **是否启用文档曲面展平**。默认值：`False`。<br>对弯曲的书页、卷曲的纸张进行平整化处理。 | 需要处理**弯曲书页或纸张**时开启；对于屏幕截图或平整的发票等文档，保持 `False` 以避免不必要的性能开销。 |
|                    | `use_textline_orientation`                                 | **是否启用文本行方向分类**。默认值：`True`。<br>检测并纠正单行文本中上下颠倒的文字。 | 如果确认处理的图片中基本没有上下颠倒的文字，可设为 `False` 以**提升约10%的处理速度**。 |
| **模型与配置路径** | `paddlex_config`                                           | **指定自定义配置文件的路径**。默认值：无。<br>使用你自己导出并修改的YAML配置文件来初始化模型。 | 当你进行了**自定义模型训练**或需要**完全离线部署**，希望加载本地模型时使用。 |
|                    | `text_detection_model_dir`<br>`text_recognition_model_dir` | **指定文字检测/识别模型的本地目录路径**。默认值：`None`。    | **离线部署**或使用**自己训练的模型**时，必须指定为包含模型文件（如 `model.pdmodel`）的目录路径。 |
| **性能与精度调优** | `text_det_limit_side_len`                                  | **设置检测阶段输入图像长边的最大尺寸**（像素）。默认值：`960`。 | **速度优先**：可降低至 `640`；**精度优先**（尤其对大图或小字）：可提高到 `1316` 或更高。 |
|                    | `text_recognition_batch_size`                              | **识别阶段的批处理大小**。默认值：`6`。                      | **GPU显存充足**（如RTX3060以上）可增大至 `16` 以提升吞吐量；**使用CPU推理**时建议设为 `1-4`。 |
|                    | `text_det_thresh`<br>`text_det_box_thresh`                 | **控制文本检测的敏感度**。前者是热力图阈值，后者是框置信度阈值。 | **漏检较多**时，可适当**降低**阈值（如0.2）；**误检较多**时，可适当**提高**阈值（如0.4）。 |
|                    | `text_det_unclip_ratio`                                    | **控制检测框的外扩程度**。默认值：`1.5`。                    | 发现文字**被截断**时，可增大至 `1.8`；发现文本框**过大**包含过多背景时，可减小至 `1.3`。 |

### 3. 模型文件管理

- **默认存储路径**：首次运行，模型文件默认会下载并保存在 `$HOME/.paddleocr/` 目录下（例如，在Windows系统中，通常是 `C:\Users\[您的用户名]\.paddleocr\`）。

- **指定其他路径**：如果您希望通过自定义配置文件来指定模型的存放位置，可以按照以下步骤操作：

  1. **导出默认配置**：首先，运行一段Python代码，将默认的配置导出为一个YAML文件。

     ```python
     from paddleocr import PPStructureV3
     ocr = PPStructureV3()
     ocr.export_paddlex_config_to_yaml("PPStructureV3_test_config.yaml")
     ```

  2. **修改模型路径**：用文本编辑器打开导出的 `PPStructureV3_test_config.yaml` 文件，找到各个模块（如文本检测、文本识别、版面分析等）配置项下的 `model_dir` 字段，将其值修改为您希望存放模型的**本地目录路径**。

  3. **使用新配置**：在初始化PPStructureV3时，通过参数加载您修改后的配置文件。

     ```python
     ocr_pipeline = PPStructureV3(paddlex_config="./PPStructureV3_test_config.yaml")
     ```
### 4. PPStructureV3配置文件详解

```yaml
SubModules:
  ChartRecognition:
    batch_size: 1
    model_dir: null
    model_name: PP-Chart2Table
    module_name: chart_recognition
  LayoutDetection:
    batch_size: 8
    layout_merge_bboxes_mode:
      0: large
      1: large
      2: union
      3: union
      4: union
      5: union
      6: union
      7: large
      8: union
      9: union
      10: union
      11: union
      12: union
      13: union
      14: union
      15: union
      16: large
      17: union
      18: union
      19: union
    layout_nms: true
    layout_unclip_ratio:
    - 1.0
    - 1.0
    model_dir: null
    model_name: PP-DocLayout_plus-L
    module_name: layout_detection
    threshold:
      0: 0.3
      1: 0.5
      2: 0.4
      3: 0.5
      4: 0.5
      5: 0.5
      6: 0.5
      7: 0.3
      8: 0.5
      9: 0.5
      10: 0.5
      11: 0.5
      12: 0.5
      13: 0.5
      14: 0.5
      15: 0.45
      16: 0.5
      17: 0.5
      18: 0.5
      19: 0.5
  RegionDetection:
    layout_merge_bboxes_mode: small
    layout_nms: true
    model_dir: null
    model_name: PP-DocBlockLayout
    module_name: layout_detection
SubPipelines:
  DocPreprocessor:
    SubModules:
      DocOrientationClassify:
        batch_size: 8
        model_dir: null
        model_name: PP-LCNet_x1_0_doc_ori
        module_name: doc_text_orientation
      DocUnwarping:
        model_dir: null
        model_name: UVDoc
        module_name: image_unwarping
    batch_size: 8
    pipeline_name: doc_preprocessor
    use_doc_orientation_classify: false
    use_doc_unwarping: false
  FormulaRecognition:
    SubModules:
      FormulaRecognition:
        batch_size: 8
        model_dir: null
        model_name: PP-FormulaNet_plus-L
        module_name: formula_recognition
    batch_size: 8
    pipeline_name: formula_recognition
    use_doc_preprocessor: false
    use_layout_detection: false
  GeneralOCR:
    SubModules:
      TextDetection:
        box_thresh: 0.6
        limit_side_len: 736
        limit_type: min
        max_side_limit: 4000
        model_dir: null
        model_name: PP-OCRv5_server_det
        module_name: text_detection
        thresh: 0.3
        unclip_ratio: 1.5
      TextLineOrientation:
        batch_size: 8
        model_dir: null
        model_name: PP-LCNet_x1_0_textline_ori
        module_name: textline_orientation
      TextRecognition:
        batch_size: 8
        model_dir: null
        model_name: PP-OCRv5_server_rec
        module_name: text_recognition
        score_thresh: 0.0
    batch_size: 8
    pipeline_name: OCR
    text_type: general
    use_doc_preprocessor: false
    use_textline_orientation: true
  SealRecognition:
    SubPipelines:
      SealOCR:
        SubModules:
          TextDetection:
            box_thresh: 0.6
            limit_side_len: 736
            limit_type: min
            max_side_limit: 4000
            model_dir: null
            model_name: PP-OCRv4_server_seal_det
            module_name: seal_text_detection
            thresh: 0.2
            unclip_ratio: 0.5
          TextRecognition:
            batch_size: 8
            model_dir: null
            model_name: PP-OCRv5_server_rec
            module_name: text_recognition
            score_thresh: 0
        batch_size: 8
        pipeline_name: OCR
        text_type: seal
        use_doc_preprocessor: false
        use_textline_orientation: false
    batch_size: 8
    pipeline_name: seal_recognition
    use_doc_preprocessor: false
    use_layout_detection: false
  TableRecognition:
    SubModules:
      TableClassification:
        model_dir: null
        model_name: PP-LCNet_x1_0_table_cls
        module_name: table_classification
      TableOrientationClassify:
        model_dir: null
        model_name: PP-LCNet_x1_0_doc_ori
        module_name: doc_text_orientation
      WiredTableCellsDetection:
        model_dir: null
        model_name: RT-DETR-L_wired_table_cell_det
        module_name: table_cells_detection
      WiredTableStructureRecognition:
        model_dir: null
        model_name: SLANeXt_wired
        module_name: table_structure_recognition
      WirelessTableCellsDetection:
        model_dir: null
        model_name: RT-DETR-L_wireless_table_cell_det
        module_name: table_cells_detection
      WirelessTableStructureRecognition:
        model_dir: null
        model_name: SLANet_plus
        module_name: table_structure_recognition
    SubPipelines:
      GeneralOCR:
        SubModules:
          TextDetection:
            box_thresh: 0.4
            limit_side_len: 736
            limit_type: min
            max_side_limit: 4000
            model_dir: null
            model_name: PP-OCRv5_server_det
            module_name: text_detection
            thresh: 0.3
            unclip_ratio: 1.5
          TextLineOrientation:
            batch_size: 8
            model_dir: null
            model_name: PP-LCNet_x1_0_textline_ori
            module_name: textline_orientation
          TextRecognition:
            batch_size: 8
            model_dir: null
            model_name: PP-OCRv5_server_rec
            module_name: text_recognition
        pipeline_name: OCR
        score_thresh: 0.0
        text_type: general
        use_doc_preprocessor: false
        use_textline_orientation: true
    pipeline_name: table_recognition_v2
    use_doc_preprocessor: false
    use_layout_detection: false
    use_ocr_model: false
batch_size: 8
format_block_content: false
pipeline_name: PP-StructureV3
use_chart_recognition: false
use_doc_preprocessor: false
use_formula_recognition: true
use_region_detection: true
use_seal_recognition: false
use_table_recognition: true
```

#### 全局配置参数

```yaml
batch_size: 8                    # 全局批处理大小
format_block_content: false      # 是否格式化块内容
pipeline_name: PP-StructureV3    # 管道名称
use_chart_recognition: false     # 是否启用图表识别
use_doc_preprocessor: false      # 是否启用文档预处理
use_formula_recognition: true    # 是否启用公式识别
use_region_detection: true       # 是否启用区域检测
use_seal_recognition: false      # 是否启用印章识别
use_table_recognition: true      # 是否启用表格识别
```

#### 子模块配置 (SubModules)

##### 1. ChartRecognition (图表识别)

```yaml
ChartRecognition:
  batch_size: 1
  model_dir: null                # 模型目录（null表示使用默认）
  model_name: PP-Chart2Table     # 图表转表格模型
  module_name: chart_recognition
```

##### 2. LayoutDetection (版面检测)

```yaml
LayoutDetection:
  batch_size: 8
  layout_merge_bboxes_mode:      # 不同类别边界框合并模式
    0: large    # 文本
    1: large    # 标题
    2: union    # 图片
    # ... 其他类别
  layout_nms: true               # 是否启用非极大值抑制
  layout_unclip_ratio: [1.0, 1.0] # 边界框扩展比例
  model_dir: null
  model_name: PP-DocLayout_plus-L # 增强版文档版面检测模型
  module_name: layout_detection
  threshold:                     # 各类别检测阈值
    0: 0.3      # 文本阈值
    1: 0.5      # 标题阈值
    2: 0.4      # 图片阈值
    # ... 其他类别阈值
```

##### 3. RegionDetection (区域检测)

```yaml
RegionDetection:
  layout_merge_bboxes_mode: small  # 小区域合并模式
  layout_nms: true
  model_dir: null
  model_name: PP-DocBlockLayout   # 文档块布局模型
  module_name: layout_detection
```

#### 子管道配置 (SubPipelines)

##### 1. DocPreprocessor (文档预处理)

```yaml
DocPreprocessor:
  SubModules:
    DocOrientationClassify:       # 文档方向分类
      batch_size: 8
      model_dir: null
      model_name: PP-LCNet_x1_0_doc_ori  # 轻量级方向分类模型
      module_name: doc_text_orientation
    DocUnwarping:                 # 文档曲面矫正
      model_dir: null
      model_name: UVDoc           # UV文档矫正模型
      module_name: image_unwarping
  use_doc_orientation_classify: false  # 是否启用方向分类
  use_doc_unwarping: false       # 是否启用曲面矫正
```

##### 2. GeneralOCR (通用OCR)

```yaml
GeneralOCR:
  SubModules:
    TextDetection:                # 文本检测
      box_thresh: 0.6            # 框置信度阈值
      limit_side_len: 736        # 图像边长限制
      limit_type: min            # 限制类型（最小边）
      max_side_limit: 4000       # 最大边限制
      model_dir: null
      model_name: PP-OCRv5_server_det  # OCRv5服务器版检测模型
      thresh: 0.3                # 热力图阈值
      unclip_ratio: 1.5          # 边界框扩展比例
    TextLineOrientation:          # 文本行方向
      batch_size: 8
      model_dir: null
      model_name: PP-LCNet_x1_0_textline_ori
      module_name: textline_orientation
    TextRecognition:             # 文本识别
      batch_size: 8
      model_dir: null
      model_name: PP-OCRv5_server_rec  # OCRv5服务器版识别模型
      score_thresh: 0.0          # 识别得分阈值
  use_textline_orientation: true # 启用文本行方向校正
```

##### 3. TableRecognition (表格识别)

```yaml
TableRecognition:
  SubModules:
    TableClassification:          # 表格分类
      model_dir: null
      model_name: PP-LCNet_x1_0_table_cls
      module_name: table_classification
    TableOrientationClassify:     # 表格方向分类
      model_dir: null
      model_name: PP-LCNet_x1_0_doc_ori
      module_name: doc_text_orientation
    WiredTableCellsDetection:     # 有线表格单元格检测
      model_dir: null
      model_name: RT-DETR-L_wired_table_cell_det  # RT-DETR模型
      module_name: table_cells_detection
    WiredTableStructureRecognition: # 有线表格结构识别
      model_dir: null
      model_name: SLANeXt_wired   # SLANeXt模型
      module_name: table_structure_recognition
    WirelessTableCellsDetection:  # 无线表格单元格检测
      model_dir: null
      model_name: RT-DETR-L_wireless_table_cell_det
      module_name: table_cells_detection
    WirelessTableStructureRecognition: # 无线表格结构识别
      model_dir: null
      model_name: SLANet_plus     # SLANet增强版
      module_name: table_structure_recognition
  use_ocr_model: false           # 是否使用OCR模型
```

##### 4. FormulaRecognition (公式识别)

```yaml
FormulaRecognition:
  SubModules:
    FormulaRecognition:
      batch_size: 8
      model_dir: null
      model_name: PP-FormulaNet_plus-L  # 公式识别增强模型
      module_name: formula_recognition
```

##### 5. SealRecognition (印章识别)

```yaml
SealRecognition:
  SubPipelines:
    SealOCR:                     # 印章OCR专用管道
      SubModules:
        TextDetection:
          model_name: PP-OCRv4_server_seal_det  # 印章专用检测模型
          thresh: 0.2           # 较低阈值适应印章
          unclip_ratio: 0.5     # 较小扩展比例
        TextRecognition:
          model_name: PP-OCRv5_server_rec
          score_thresh: 0       # 无得分阈值
      use_textline_orientation: false  # 印章不需要方向校正
```

## 第三部分：PaddleOCR-VL多模态文档解析

### 1. PaddleOCR-VL简介

#### 百度飞桨在2025年下半年推出的一个多模态文档智能解析模型，它在传统的文字识别（OCR）基础上，融合了视觉与语言模型的理解能力，能同时处理文本、表格、公式和图表等多种元素。

| 模型名称         | 发布机构/团队 | 参数规模       | 核心特点                                                     | 主要应用场景                                       |
| :--------------- | :------------ | :------------- | :----------------------------------------------------------- | :------------------------------------------------- |
| **PaddleOCR-VL** | 百度          | 0.9B           | 两阶段处理，先分析版面布局再识别内容；在权威评测中综合性能领先 | 复杂排版的文档解析、多语言文本识别、表格和公式提取 |
| **DeepSeek-OCR** | DeepSeek      | 约3B (MoE架构) | 创新的视觉压缩技术，将图像信息高效压缩为少量视觉Token，处理长文档效率高 | 长文档、书籍的高效处理；为大型模型准备训练数据     |
| **Qwen2.5-VL**   | 阿里巴巴      | 3B/7B/72B      | 通用型视觉语言模型，能力均衡，支持长视频理解、视觉智能体操控等复杂任务 | 视觉问答、图文理解、结构化数据输出、智能体应用     |

#### PaddleOCR-VL的核心技术：两阶段处理流程

PaddleOCR-VL的卓越表现，很大程度上归功于其巧妙的两阶段处理流程。这让它在参数量不大的情况下，实现了极高的准确率。

1.  **第一阶段：专业化版面分析**
    -   首先，一个名为**PP-DocLayoutV2**的专用视觉模型会像"侦察兵"一样快速扫描整个文档图像。
    -   它的任务非常纯粹：**进行布局检测**，将文档中不同属性的区域（如标题、正文、表格、公式、图片）用框标出来，并确定符合人类习惯的**阅读顺序**。

2.  **第二阶段：分而治之的结构化识别**
    -   接着，核心的PaddleOCR-VL模型（0.9B参数）才登场。但它面对的已经不是复杂的整张A4纸，而是上一阶段裁剪好的、**一个个被标注了类型的小图片**。
    -   它的任务也变得非常专注：根据图片类型进行识别。例如，收到"表格"小图，就把它转成Markdown；收到"公式"小图，就把它转成LaTeX。这种"分而治之"的策略，极大地降低了模型的理解难度。

### 2. PaddleOCR-VL安装与使用

#### 请注意安装 3.2.1 及以上版本的飞桨框架，同时安装特殊版本的 safetensors。

下载：[https://xly-devops.cdn.bcebos.com/safetensors-nightly/safetensors-0.6.2.dev0-cp38-abi3-win_amd64.whl](https://xly-devops.cdn.bcebos.com/safetensors-nightly/safetensors-0.6.2.dev0-cp38-abi3-win_amd64.whl)

参考：[PaddleOCR-VL · 模型库](https://modelscope.cn/models/PaddlePaddle/PaddleOCR-VL/feedback/issueDetail/48499)

PaddleOCR-VL推理代码如下：

```python
from paddleocr import PaddleOCRVL

pipeline = PaddleOCRVL()
# pipeline = PaddleOCRVL(use_doc_orientation_classify=True) # 通过 use_doc_orientation_classify 指定是否使用文档方向分类模型
# pipeline = PaddleOCRVL(use_doc_unwarping=True) # 通过 use_doc_unwarping 指定是否使用文本图像矫正模块
# pipeline = PaddleOCRVL(use_layout_detection=False) # 通过 use_layout_detection 指定是否使用版面区域检测排序模块
output = pipeline.predict("./paddleocr_vl_demo.png")
for res in output:
    res.print() ## 打印预测的结构化输出
    res.save_to_json(save_path="output") ## 保存当前图像的结构化json结果
    res.save_to_markdown(save_path="output") ## 保存当前图像的markdown格式的结果
```

如果是 PDF 文件，会将 PDF 的每一页单独处理，每一页的 Markdown 文件也会对应单独的结果。如果希望整个 PDF 文件转换为 Markdown 文件，建议使用以下的方式运行：

```python
from pathlib import Path
from paddleocr import PaddleOCRVL

input_file = "./your_pdf_file.pdf"
output_path = Path("./output")

pipeline = PaddleOCRVL()
output = pipeline.predict(input=input_file)

markdown_list = []
markdown_images = []

for res in output:
    md_info = res.markdown
    markdown_list.append(md_info)
    markdown_images.append(md_info.get("markdown_images", {}))

markdown_texts = pipeline.concatenate_markdown_pages(markdown_list)

mkd_file_path = output_path / f"{Path(input_file).stem}.md"
mkd_file_path.parent.mkdir(parents=True, exist_ok=True)

with open(mkd_file_path, "w", encoding="utf-8") as f:
    f.write(markdown_texts)

for item in markdown_images:
    if item:
        for path, image in item.items():
            file_path = output_path / path
            file_path.parent.mkdir(parents=True, exist_ok=True)
            image.save(file_path)
```
在上述 Python 脚本中，执行了如下几个步骤：

### 3. PaddleOCR-VL配置文件详解

```yaml
SubModules:
  LayoutDetection:
    batch_size: 8
    layout_merge_bboxes_mode:
      0: union
      1: union
      2: union
      3: large
      4: union
      5: large
      6: large
      7: union
      8: union
      9: union
      10: union
      11: union
      12: union
      13: union
      14: union
      15: large
      16: union
      17: large
      18: union
      19: union
      20: union
      21: union
      22: union
      23: union
      24: union
    layout_nms: true
    layout_unclip_ratio:
    - 1.0
    - 1.0
    model_dir: "D:\\official_models\\PP-DocLayoutV2"
    model_name: PP-DocLayoutV2
    module_name: layout_detection
    threshold:
      0: 0.5
      1: 0.5
      2: 0.5
      3: 0.5
      4: 0.5
      5: 0.4
      6: 0.4
      7: 0.5
      8: 0.5
      9: 0.5
      10: 0.5
      11: 0.5
      12: 0.5
      13: 0.5
      14: 0.5
      15: 0.4
      16: 0.5
      17: 0.4
      18: 0.5
      19: 0.5
      20: 0.45
      21: 0.5
      22: 0.4
      23: 0.4
      24: 0.5
  VLRecognition:
    batch_size: 4096
    genai_config:
      backend: native
    model_dir: "D:\\official_models\\PaddleOCR-VL"
    model_name: PaddleOCR-VL-0.9B
    module_name: vl_recognition
SubPipelines:
  DocPreprocessor:
    SubModules:
      DocOrientationClassify:
        batch_size: 8
        model_dir: null
        model_name: PP-LCNet_x1_0_doc_ori
        module_name: doc_text_orientation
      DocUnwarping:
        model_dir: null
        model_name: UVDoc
        module_name: image_unwarping
    batch_size: 8
    pipeline_name: doc_preprocessor
    use_doc_orientation_classify: true
    use_doc_unwarping: true
batch_size: 64
format_block_content: false
pipeline_name: PaddleOCR-VL
use_chart_recognition: false
use_doc_preprocessor: false
use_layout_detection: true
use_queues: true
```

#### 🏗️ 整体架构概览

```yaml
pipeline_name: PaddleOCR-VL          # 管道名称：PaddleOCR-VL多模态系统
batch_size: 64                       # 全局批处理大小（较大，适合VL模型）
use_layout_detection: true           # 启用版面检测（第一阶段）
use_doc_preprocessor: false          # 禁用文档预处理（VL模型自身能力强）
use_queues: true                     # 启用队列处理，提高吞吐量
```

#### 🔍 核心子模块配置 (SubModules)

##### 1. LayoutDetection (版面检测 - 第一阶段)

这是PaddleOCR-VL两阶段流程中的**第一阶段**，负责文档结构的初步分析。

```yaml
LayoutDetection:
  model_dir: "D:\\official_models\\PP-DocLayoutV2"
  model_name: PP-DocLayoutV2          # 专用版面分析模型V2版本
  batch_size: 8
  layout_nms: true                    # 启用非极大值抑制，去除重复框
```

**版面类别与阈值配置**：

- **25个语义类别**：相比PPStructureV3的20类，新增了5个更细粒度的文档元素类别
- **智能阈值策略**：不同类别使用不同检测阈值
  - 大部分文本区域：`0.5`（较高置信度）
  - 复杂元素（类别5,6,15,17,22,23）：`0.4`（稍低阈值，避免漏检）
  - 特殊元素（类别20）：`0.45`（中间阈值）

**边界框合并策略**：

```yaml
layout_merge_bboxes_mode:
  0: union    # 文本区域 - 合并模式
  1: union    # 标题 - 合并模式  
  2: union    # 图片 - 合并模式
  3: large    # 表格 - 取大模式（保留完整表格结构）
  5: large    # 页眉 - 取大模式
  6: large    # 页脚 - 取大模式
  15: large   # 公式 - 取大模式（确保公式完整性）
  17: large   # 图表 - 取大模式
```

- **union模式**：合并重叠区域，适合文本类连续内容
- **large模式**：保留最大边界框，确保特殊元素（表格、公式等）的完整性

##### 2. VLRecognition (视觉语言识别 - 第二阶段)

这是PaddleOCR-VL的**核心模块**，负责基于版面分析结果进行精细化内容识别。

```yaml
VLRecognition:
  model_dir: "D:\\official_models\\PaddleOCR-VL"
  model_name: PaddleOCR-VL-0.9B        # 0.9B参数的多模态模型
  batch_size: 4096                     # 极大批处理大小（利用VL模型高效性）
  genai_config:
    backend: native                    # 使用原生推理后端
```

**关键特点**：

- **专用模型路径**：明确指向PaddleOCR-VL模型目录
- **超大batch_size**：`4096`说明VL模型在处理裁剪后的小图时极其高效
- **原生推理**：优化过的推理后端，确保最佳性能

#### ⚙️ 预处理管道配置 (SubPipelines)

##### DocPreprocessor (文档预处理)

虽然全局关闭(`use_doc_preprocessor: false`)，但配置中保留了完整的预处理能力：

```yaml
DocPreprocessor:
  use_doc_orientation_classify: true    # 文档方向校正
  use_doc_unwarping: true              # 文档曲面矫正
  SubModules:
    DocOrientationClassify:
      model_name: PP-LCNet_x1_0_doc_ori  # 轻量级方向分类模型
    DocUnwarping:
      model_name: UVDoc                 # UV文档展平模型
```
### 4.PaddleOCR-VL API
**在线API参考：**[https://ai.baidu.com/ai-doc/AISTUDIO/2mh4okm66](https://ai.baidu.com/ai-doc/AISTUDIO/2mh4okm66)
```python
import base64
import urllib
import requests
import time
import os
import glob

API_KEY = "******"
SECRET_KEY = "*************"


def submit_task(image_path):
    """提交图片解析任务"""
    url = "https://aip.baidubce.com/rest/2.0/brain/online/v2/paddle-vl-parser/task?access_token=" + get_access_token()
    file_name = image_path.split("/")[-1] if "/" in image_path else image_path.split("\\")[-1]
    file_data = get_file_content_as_base64(image_path, True)

    payload = f'file_data={file_data}&file_name={file_name}&analysis_chart=False'
    headers = {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Accept': 'application/json'
    }

    response = requests.request("POST", url, headers=headers, data=payload.encode("utf-8"))
    response.encoding = "utf-8"

    if response.status_code == 200:
        result = response.json()
        if result.get("error_code") == 0:
            task_id = result["result"]["task_id"]
            print(f"任务提交成功，任务ID: {task_id}")
            return task_id
        else:
            print(f"任务提交失败: {result.get('error_msg')}")
            return None
    else:
        print(f"请求失败，状态码: {response.status_code}")
        return None


def query_task_result(task_id, max_retries=10, delay=2):
    """查询任务结果，支持重试机制"""
    url = "https://aip.baidubce.com/rest/2.0/brain/online/v2/paddle-vl-parser/task/query?access_token=" + get_access_token()

    payload = f'task_id={task_id}'
    headers = {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Accept': 'application/json'
    }

    for i in range(max_retries):
        print(f"第 {i + 1} 次查询任务结果...")
        response = requests.request("POST", url, headers=headers, data=payload.encode("utf-8"))
        response.encoding = "utf-8"

        if response.status_code == 200:
            result = response.json()

            # 检查任务是否完成
            if result.get("error_code") == 0:
                task_status = result["result"].get("status")

                if task_status == "success":
                    print("任务处理完成!")
                    return result
                elif task_status == "failed":
                    print("任务处理失败!")
                    return result
                else:
                    print(f"任务处理中，当前状态: {task_status}")
            else:
                print(f"查询失败: {result.get('error_msg')}")

        # 如果不是最后一次重试，则等待
        if i < max_retries - 1:
            print(f"等待 {delay} 秒后重试...")
            time.sleep(delay)

    print(f"经过 {max_retries} 次重试后仍未获取到结果")
    return None


def download_and_save_files(result, image_path, output_dir="output_api"):
    """下载并保存解析结果和Markdown文件到指定输出目录"""
    try:
        # 确保输出目录存在
        if not os.path.exists(output_dir):
            os.makedirs(output_dir)
            print(f"创建输出目录: {output_dir}")

        # 获取图片名称（不带扩展名）
        image_name = os.path.splitext(os.path.basename(image_path))[0]

        # 下载解析结果JSON文件
        parse_result_url = result["result"]["parse_result_url"]
        parse_response = requests.get(parse_result_url)
        if parse_response.status_code == 200:
            json_filename = os.path.join(output_dir, f"{image_name}.json")
            with open(json_filename, 'w', encoding='utf-8') as f:
                f.write(parse_response.text)
            print(f"解析结果已保存为: {json_filename}")
        else:
            print(f"下载解析结果失败，状态码: {parse_response.status_code}")

        # 下载Markdown文件
        markdown_url = result["result"]["markdown_url"]
        markdown_response = requests.get(markdown_url)
        if markdown_response.status_code == 200:
            md_filename = os.path.join(output_dir, f"{image_name}.md")
            with open(md_filename, 'w', encoding='utf-8') as f:
                f.write(markdown_response.text)
            print(f"Markdown文件已保存为: {md_filename}")
        else:
            print(f"下载Markdown文件失败，状态码: {markdown_response.status_code}")

    except Exception as e:
        print(f"保存文件时出错: {e}")


def get_file_content_as_base64(path, urlencoded=False):
    """
    获取文件base64编码
    :param path: 文件路径
    :param urlencoded: 是否对结果进行urlencoded
    :return: base64编码信息
    """
    with open(path, "rb") as f:
        content = base64.b64encode(f.read()).decode("utf8")
        if urlencoded:
            content = urllib.parse.quote_plus(content)
    return content


def get_access_token():
    """
    使用 AK，SK 生成鉴权签名（Access Token）
    :return: access_token，或是None(如果错误)
    """
    url = "https://aip.baidubce.com/oauth/2.0/token"
    params = {"grant_type": "client_credentials", "client_id": API_KEY, "client_secret": SECRET_KEY}
    return str(requests.post(url, params=params).json().get("access_token"))


def process_single_image(image_path, output_dir="output_api"):
    """处理单张图片的完整流程"""
    print(f"\n开始处理图片: {image_path}")

    # 提交任务
    task_id = submit_task(image_path)

    if task_id:
        # 查询结果
        result = query_task_result(task_id)
        if result:
            print("任务处理成功!")
            # 下载并保存文件到指定输出目录
            download_and_save_files(result, image_path, output_dir)
            return True
        else:
            print(f"未能获取到图片 {image_path} 的处理结果")
            return False
    else:
        print(f"图片 {image_path} 任务提交失败，无法查询结果")
        return False


def main():
    input_folder = "images"
    output_folder = "output"

    # 检查输入文件夹是否存在
    if not os.path.exists(input_folder):
        print(f"错误: 输入文件夹 '{input_folder}' 不存在!")
        return

    # 获取picture文件夹中的所有图片文件
    image_extensions = ['*.jpg', '*.jpeg', '*.png', '*.bmp', '*.gif', '*.tiff']
    image_files = []

    for extension in image_extensions:
        image_files.extend(glob.glob(os.path.join(input_folder, extension)))
        # image_files.extend(glob.glob(os.path.join(input_folder, extension.upper())))

    if not image_files:
        print(f"在文件夹 '{input_folder}' 中未找到图片文件!")
        return

    print(f"找到 {len(image_files)} 个图片文件:")
    for img in image_files:
        print(f"  - {img}")

    # 处理每张图片
    success_count = 0
    for image_path in image_files:
        if process_single_image(image_path, output_folder):
            success_count += 1
        print("-" * 50)

    print(f"\n处理完成! 成功处理 {success_count}/{len(image_files)} 个图片文件")
    print(f"输出文件保存在 '{output_folder}' 文件夹中")


if __name__ == '__main__':
    main()
```

### 5.PaddleOCRVL 的所有参数

```python
# Copyright (c) 2025 PaddlePaddle Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

from .._utils.cli import (
    add_simple_inference_args,
    get_subcommand_args,
    perform_simple_inference,
    str2bool,
)
from .base import PaddleXPipelineWrapper, PipelineCLISubcommandExecutor
from .utils import create_config_from_structure


_SUPPORTED_VL_BACKENDS = ["native", "vllm-server", "sglang-server", "fastdeploy-server"]


class PaddleOCRVL(PaddleXPipelineWrapper):
    def __init__(
        self,
        layout_detection_model_name=None,
        layout_detection_model_dir=None,
        layout_threshold=None,
        layout_nms=None,
        layout_unclip_ratio=None,
        layout_merge_bboxes_mode=None,
        vl_rec_model_name=None,
        vl_rec_model_dir=None,
        vl_rec_backend=None,
        vl_rec_server_url=None,
        vl_rec_max_concurrency=None,
        doc_orientation_classify_model_name=None,
        doc_orientation_classify_model_dir=None,
        doc_unwarping_model_name=None,
        doc_unwarping_model_dir=None,
        use_doc_orientation_classify=None,
        use_doc_unwarping=None,
        use_layout_detection=None,
        use_chart_recognition=None,
        format_block_content=None,
        **kwargs,
    ):
        if vl_rec_backend is not None and vl_rec_backend not in _SUPPORTED_VL_BACKENDS:
            raise ValueError(
                f"Invalid backend for the VL recognition module: {vl_rec_backend}. Supported values are {_SUPPORTED_VL_BACKENDS}."
            )

        params = locals().copy()
        params.pop("self")
        params.pop("kwargs")
        self._params = params

        super().__init__(**kwargs)

    @property
    def _paddlex_pipeline_name(self):
        return "PaddleOCR-VL"

    def predict_iter(
        self,
        input,
        *,
        use_doc_orientation_classify=None,
        use_doc_unwarping=None,
        use_layout_detection=None,
        use_chart_recognition=None,
        layout_threshold=None,
        layout_nms=None,
        layout_unclip_ratio=None,
        layout_merge_bboxes_mode=None,
        use_queues=None,
        prompt_label=None,
        format_block_content=None,
        repetition_penalty=None,
        temperature=None,
        top_p=None,
        min_pixels=None,
        max_pixels=None,
        **kwargs,
    ):
        return self.paddlex_pipeline.predict(
            input,
            use_doc_orientation_classify=use_doc_orientation_classify,
            use_doc_unwarping=use_doc_unwarping,
            use_layout_detection=use_layout_detection,
            use_chart_recognition=use_chart_recognition,
            layout_threshold=layout_threshold,
            layout_nms=layout_nms,
            layout_unclip_ratio=layout_unclip_ratio,
            layout_merge_bboxes_mode=layout_merge_bboxes_mode,
            use_queues=use_queues,
            prompt_label=prompt_label,
            format_block_content=format_block_content,
            repetiti

