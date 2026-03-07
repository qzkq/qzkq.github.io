---
title: Ollama详解，无网环境导入运行本地下载的大模型，无网环境pycharm插件大模型调用、Ollama Python api、coze-studio
date: 2026-01-01 08:19:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Ollama详解，无网环境导入运行本地下载的大模型，无网环境pycharm插件大模型调用、Ollama Python api、coze-studio)
![在这里插入图片描述](/img/739dd679474c4266b996719b46fa0081.png)

# ollama
[Ollama](https://ollama.com/download) 是一个开源的大型语言模型服务工具，旨在帮助用户快速在本地运行大模型。通过简单的安装指令，用户可以通过一条命令轻松启动和运行开源的大型语言模型。 它提供了一个简洁易用的命令行界面和服务器，专为构建大型语言模型应用而设计。用户可以轻松下载、运行和管理各种开源 LLM。与传统 LLM 需要复杂配置和强大硬件不同，Ollama 能够让用户在消费级的 PC 上体验 LLM 的强大功能。

Ollama 会自动监测本地计算资源，如有 GPU 的条件，会优先使用 GPU 的资源，同时模型的推理速度也更快。如果没有 GPU 条件，直接使用 CPU 资源。

Ollama 极大地简化了在 Docker 容器中部署和管理大型语言模型的过程，使用户能够迅速在本地启动和运行这些模型。

Ollama 支持的模型库列表 [https://ollama.com/library](https://ollama.com/library)

注意：运行 7B 模型至少需要 8GB 内存，运行 13B 模型至少需要 16GB 内存，运行 33B 模型至少需要 32GB 内存。
## 环境变量配置
![在这里插入图片描述](/img/48c6b6b67aa24918bb96d6010439e502.png)
### 更改模型存储位置：设置 OLLAMA_MODELS 环境变量
![在这里插入图片描述](/img/d12690cfea754224836c6233e1553607.png)
### 修改监听地址：设置 OLLAMA_HOST 环境变量

 - 0.0.0.0 表示监听所有网络接口，允许局域网访问。 
 - 11434 是 Ollama 默认端口。

![在这里插入图片描述](/img/9125484b6a924cbf965a5b153216bd6e.png)

验证环境变量是否生效 `echo %OLLAMA_MODELS%`

### Ollama 自定义在 GPU 中运行
Ollama 默认情况下使用 CPU 进行推理。为了获得更快的推理速度，可以配置 Ollama 使用的 GPU。
#### 前提条件

 1. 电脑有 NVIDIA 显卡。
 2. 已安装 NVIDIA 显卡驱动程序，可以使用命令 `nvidia-smi` 来检查驱动程序是否安装。
 3. 已安装 CUDA 工具包，可以使用命令 `nvcc --version` 来检查 CUDA 是否安装。
![在这里插入图片描述](/img/0cc0e9bd539e40d3a887a88e96f8a682.png)
####  指定使用的 GPU
如果你的系统有多个 GPU，并且你想指定 Ollama 使用特定的 GPU，可以设置 **CUDA_VISIBLE_DEVICES** 环境变量。
 1. 查找 GPU 的 UUID： 强烈建议使用 UUID 而不是编号，因为编号可能会因为驱动更新或系统重启而发生变化。
 2. 打开命令提示符或 PowerShell。

	 - 运行命令：`nvidia-smi -L`
	 - 在输出中，找到想要使用的 GPU 的 "UUID" 值。

![在这里插入图片描述](/img/df4be33edbd049e99b001fe240fe7699.png)

 3. 创建 CUDA_VISIBLE_DEVICES 变量：

	 - 变量名： CUDA_VISIBLE_DEVICES
	 - 变量值： 找到的 GPU 的 UUID。 例如：GPU-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

4. 运行 Ollama。 新开一个命令提示符窗口，使用 `ollama ps` 命令查看 Ollama 运行的进程。

![在这里插入图片描述](/img/5abb1ad7697d4aeeb763f21474d7053e.png)

## 常用命令
![在这里插入图片描述](/img/7ed09bb9979f4b88a5e7ad789cf6b162.png)
###  ollama serve
运行ollama，提示报错
```python
Error: listen tcp 127.0.0.1:11434: bind: Only one usage of each socket address (protocol/network address/port) is normally permitted.
```

解决方法：禁用自启，结束进程，重启服务。

![在这里插入图片描述](/img/84ac5f99b1684570a97b341bbd06befa.png)
![在这里插入图片描述](/img/bb4255d422d149859e237d563a7212f2.png)
## 本地下载的gguf如何导入ollama

1.`ollama`安装是否成功

![在这里插入图片描述](/img/52b8cf5a40c94798968b9cebd3c432c2.png)

2.将model放在文件夹下，创建一个txt配置文件，如下所示：

![在这里插入图片描述](/img/70464560271c48bc8390aab0f929bd24.png)
3. 打开cmd，导航到你的配置文件所在路径； 在cmd中输入以下命令：`ollama create 模型的名字 -f config.txt`。
4. 等待ollama完成模型的创建和部署；使用`ollama list`验证模型是否部署完成。
![在这里插入图片描述](/img/1582c6046a5641349a7126beb8ea540e.png)
5. 使用`ollama run qwen2:0.5b`运行。
多行输入命令时，可以使用 """ 进行换行；使用 """ 结束换行。终止 Ollama 模型推理服务，可以使用 `/bye`。
![在这里插入图片描述](/img/39c44b23be3c48f1b0a90ff02da22d2d.png)
注意：Ollama 进程会一直运行，如果需要终止 Ollama 所有相关进程，可以使用以下命令：

```python
Get-Process | Where-Object {$_.ProcessName -like '*ollama*'} | Stop-Process
```

## 如何将Hugging Face的SafeTensors模型转换为GGUF格式，以便在ollama平台上运行

Safetensors 是一种用于存储深度学习模型权重的文件格式，它旨在解决安全性、效率和易用性方面的问题。

1.从[https://opencsg.com/models](https://opencsg.com/models)下载模型。

2.使用 `llama.cpp` [https://github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) 进行转换。llama.cpp 是 GGML 主要作者基于最早的 llama 的 c/c++ 版本开发的，目的就是希望用 CPU 来推理各种 LLM。克隆 llama.cpp 库到本地，与下载的模型放在同一目录下。

```python
git clone https://github.com/ggerganov/llama.cpp.git
```
3.使用 `llama.cpp` 转换模型的流程基于 python 开发，需要安装相关的库，推荐使用 conda 或 venv 新建一个环境。

```python
cd llama.cpp
pip install -r requirements.txt
python convert_hf_to_gguf.py -h
```
![在这里插入图片描述](/img/f9091e7375f84bf680bd90e349624e72.png)

```python
python convert_hf_to_gguf.py ../Qwen2-1.5B-Instruct --outfile Qwen2_instruct_1.5b.gguf --outtype f16
```

![在这里插入图片描述](/img/79785a5567554436bc7c67e0ec39ea80.png)
![在这里插入图片描述](/img/180e7b8a2c7742d2b14a22371b36b62a.png)

4.使用`llama.cpp`进行模型量化

模型量化是一种技术，将高精度的浮点数模型转换为低精度模型，模型量化的主要目的是减少模型的大小和计算成本，尽可能保持模型的准确性，其目标是使模型能够在资源有限的设备上运行，例如CPU或者移动设备。

 - 创建 Modelfile 文件，编写以下内容：`FROM ./Qwen2_instruct_1.5b.gguf`

 - `ollama create -q Q4_K_M mymodel3 -f ./Modelfile`（Q4_K_M 是一种 4 位（4-bit）量化格式。）

![在这里插入图片描述](/img/c88ee8fee4f44b75a3a0a4ba9e402f22.png)
## 自定义 Prompt

Ollama 支持自定 义Prompt，可以让模型生成更符合用户需求的文本。

 - 根目录下创建一个 Modelfile 文件

```python
FROM llama3.1
# sets the temperature to 1 [higher is more creative, lower is more coherent]
# 设置温度为1[越高越有创意，越低越连贯]
PARAMETER temperature 1
# sets the context window size to 4096, this controls how many tokens the LLM can use as context to generate the next token
# 设置上下文窗口大小为4096，这控制了LLM可以使用多少令牌作为上下文来生成下一个令牌
PARAMETER num_ctx 4096

# sets a custom system message to specify the behavior of the chat assistant
# 设置自定义系统消息来指定聊天助手的行为
# SYSTEM You are Mario from super mario bros, acting as an assistant.
SYSTEM 你是一位在数学竞赛、数学建模竞赛、大数据竞赛以及人工智能（涵盖深度学习、大模型和机器学习）领域拥有卓越成就的专家型 AI。擅长以生动、有趣且浅显易懂的方式，为用户深入阐释相关知识，同时还能根据要求生成图像并进行说明。
```

 - 创建模型

```python
ollama create mymodel -f ./Modelfile
```
## 无网环境Pycharm插件Proxy AI调用本地大模型

[https://plugins.jetbrains.com/plugin/21056-proxy-ai](https://plugins.jetbrains.com/plugin/21056-proxy-ai)

Proxy AI 可以在无网环境下不需要登录即可使用
![在这里插入图片描述](/img/2b9a5e9c90b1497ba4ce607ea09dbc04.png)
# Ollama Python api
## 核心方法参数详解

### 1. 生成文本 (`generate()`)
```python
import ollama

response = ollama.generate(
    model='llama3',            # 必需：模型名称
    prompt='为什么天空是蓝色的？',  # 必需：提示文本
    system='你是一位科学家',      # 可选：系统角色设定
    stream=False,               # 可选：是否流式响应 (默认False)
    format='json',              # 可选：输出格式 (json/text)
    options={                   # 可选：高级参数
        'temperature': 0.7,
        'num_predict': 100,
        'top_k': 40,
        'stop': ['\n', '。']
    },
    template='''<|system|>{system}</s>
                <|user|>{prompt}</s>
                <|assistant|>''',  # 可选：自定义模板
    context=[...]              # 可选：历史上下文
)

print(response['response'])
```
####  `options` 参数详解
##### 1. 温度控制 (Temperature)

```python
options = {
    "temperature": 0.7  # 默认值通常为 0.8
}
```

- **作用**：控制输出的随机性
- **取值范围**：0.0 ~ 1.0
- **效果**：
  - 低值 (0.1-0.5)：输出更确定、保守
  - 高值 (0.7-1.0)：输出更有创意、多样化
  - 0.0：完全确定性输出（可能重复）

##### 2. 最大生成长度 (num_predict)

```python
options = {
    "num_predict": 512  # 默认值通常为 128
}
```

- **作用**：限制模型生成的最大 token 数量
- **注意**：实际输出长度可能小于此值（遇到停止词会提前终止）

##### 3. Top-P 采样 (top_p)

```python
options = {
    "top_p": 0.9  # 默认值通常为 0.9
}
```

- **作用**：控制采样范围（核采样）
- **取值范围**：0.0 ~ 1.0
- **效果**：
  - 0.9 = 只考虑累计概率达 90% 的高概率 token
  - 低值：输出更集中
  - 1.0：无限制（所有 token）

##### 4. Top-K 采样 (top_k)

```python
options = {
    "top_k": 40  # 默认值通常为 40
}
```

- **作用**：限制每步采样考虑的 token 数量
- **效果**：
  - 低值：输出更可预测
  - 高值：输出更多样化
  - 0：禁用（使用所有 token）

##### 5. 重复惩罚 (repeat_penalty)

```python
options = {
    "repeat_penalty": 1.1  # 默认值通常为 1.1
}
```

- **作用**：防止重复输出

- **取值**：

  - \>1.0：惩罚重复内容（值越大惩罚越重）

  - 1.0：无惩罚

  - <1.0：鼓励重复

##### 6. 停止序列 (stop)

```python
options = {
    "stop": ["\n", "###", "用户:"]  # 遇到这些序列时停止生成
}
```

- **作用**：定义停止生成的条件序列
- **类型**：字符串列表
- **常见用例**：停止在对话分隔符、特定标记处

##### 7. 频率惩罚 (frequency_penalty)

```python
options = {
    "frequency_penalty": 0.5
}
```

- **作用**：降低频繁出现 token 的概率

- **取值范围**：-2.0 ~ 2.0

- **效果**：

  - \>0：惩罚常见词

  - <0：鼓励常见词

##### 8. 存在惩罚 (presence_penalty)

```python
options = {
    "presence_penalty": 0.3
}
```

- **作用**：惩罚已经出现过的 token

- **取值范围**：-2.0 ~ 2.0

- **效果**：

  - \>0：鼓励新内容

  - <0：鼓励重复内容

##### 9. Mirostat 采样 (mirostat)

```python
options = {
    "mirostat": 2,      # 0=禁用, 1=Mirostat, 2=Mirostat 2.0
    "mirostat_tau": 5.0, # 目标困惑度 (默认5.0)
    "mirostat_eta": 0.1  # 学习率 (默认0.1)
}
```

- **作用**：更智能的采样方法，自动调整输出质量
- **推荐**：对于创意写作，设置 `mirostat: 2`

##### 10. 上下文窗口 (num_ctx)
上下文窗口‌是指在处理自然语言时，模型能够同时考虑和利用的上下文信息的范围。具体来说，它决定了模型在生成或理解文本时，可以同时看到和利用多少个词或字符的信息‌。
```python
options = {
    "num_ctx": 4096  # 上下文token数量
}
```

- **作用**：控制模型考虑的上下文长度
- **注意**：不能超过模型的最大上下文限制

##### 11. 批处理大小 (num_batch)

```python
options = {
    "num_batch": 512  # 并行处理的token数
}
```

- **作用**：影响生成速度
- **调整建议**：根据GPU内存调整

#### 2. 聊天对话 (`chat()`) - 推荐多轮对话
```python
response = ollama.chat(
    model='llama3',
    messages=[  # 必需：消息历史
        {'role': 'user', 'content': '你好！'},
        {'role': 'assistant', 'content': '你好，有什么可以帮助您？'},
        {'role': 'user', 'content': '量子计算是什么？'}
    ],
    stream=True,  # 流式响应示例
    options={'temperature': 0.5}
)

# 流式响应处理
for chunk in response:
    print(chunk['message']['content'], end='', flush=True)
```

##### print参数`flush`详解

flush 参数允许你强制立即刷新输出缓冲区。当设置为 `True` 时：

1. 输出内容会**立即显示**，不等待缓冲区填满
2. 绕过常规的缓冲机制
3. 确保信息实时可见

#### 3. 嵌入向量 (`embeddings()`)
Ollama 的 `embeddings` 功能是将文本转化为数值向量（嵌入向量）的核心工具，这些向量能够捕捉文本的语义信息，是构建智能应用的基础。
##### 文本嵌入是什么？

文本嵌入是将**离散文本**转换为**连续向量空间**中的数值表示：

- 每个词语/句子被映射为高维向量（通常128-2048维）
- 语义相似的文本在向量空间中位置相近
- 支持数学运算（如向量加减、相似度计算）
```python
embeddings = ollama.embeddings(
    model='nomic-embed-text',
    prompt='文本嵌入示例'
)
print(embeddings['embedding'])
```

---
### 实用管理方法

#### 1. 列出本地模型
```python
models = ollama.list()
print([model['name'] for model in models['models']])
```

#### 2. 下载模型
```python
ollama.pull('llama3')  # 下载最新版
ollama.pull('mistral:7b-instruct-q4_K_M')  # 下载特定版本
```

#### 3. 删除模型
```python
ollama.delete('llama2')
```

---

### 参数对照表（Python vs HTTP API）

| 功能               | Python 方法              | HTTP 端点              | 优势对比                     |
|--------------------|--------------------------|------------------------|------------------------------|
| 文本生成           | `generate()`             | `POST /api/generate`   | Python 自动处理 JSON 解析    |
| 聊天对话           | `chat()`                 | `POST /api/chat`       | 直接处理消息字典             |
| 获取嵌入向量       | `embeddings()`           | `POST /api/embeddings` | 返回结构化向量数据           |
| 模型管理           | `list()`/`pull()`/`delete()` | `GET /api/tags` 等     | 封装为同步方法，无需请求构造 |
| 流式响应           | 返回生成器               | 流式 HTTP 响应         | 用 `for` 循环直接迭代        |

---
# coze-studio
Coze Studio 是由字节跳动开源的一站式 AI Agent 开发平台，于 2025 年 7 月 26 日正式开源，采用 Apache 2.0 协议，支持免费商用和私有化部署。

---
## 一、核心功能与特性

1. **可视化工作流编排**  
   - 通过拖拽节点（如 LLM 调用、工具执行、条件分支）构建复杂业务逻辑，无需编码即可设计 AI Agent 的工作流。
   - 支持多模态输入输出（文本、图像、音频），例如上传图片→OCR 识别→LLM 生成回复→语音合成的端到端流程。

2. **插件生态与扩展能力**  
   - 提供开放式插件框架，支持封装第三方 API、数据库或私有工具（如日历查询、邮件发送、CRM 集成）。
   - 后续将打通商业版与开源版插件市场，支持第三方开发者上架和变现。

3. **多模型与多模态支持**  
   - 集成主流大模型（OpenAI、Qwen、Llama、GLM、火山方舟等），通过统一接口抽象实现灵活切换。
   - 内置 RAG（检索增强生成）能力，支持知识库上传与向量检索（基于 Milvus 或 Elasticsearch），提升问答准确性。

4. **全生命周期管理**  
   - 与 **Coze Loop**（配套运维平台）协同，提供 Prompt 调试、链路追踪（Trace）、多模型评测（BLEU/ROUGE 指标 + LLM 自动评分）等运维能力。

---
## 二、快速部署指南

1. **环境要求**  

   - 系统：Linux/Mac（推荐），2 核 4GB 内存。
   - 依赖：Docker 及 Docker Compose。

2. **部署步骤**  

   ```bash
   # 克隆仓库
   git clone https://github.com/coze-dev/coze-studio.git
   cd coze-studio
   
   # 配置模型 API Key（以火山引擎为例）
   cp backend/conf/model/template/model_template_ark_doubao-seed-1.6.yaml backend/conf/model/
   # 编辑 YAML 文件，填写 id、api_key、model 等字段
   
   # 启动服务
   cd docker
   cp .env.example .env
   docker compose --profile "*" up -d
   ```

3. **访问应用**  
   浏览器打开 `http://localhost:8888`，注册账号后即可创建 Agent。

![在这里插入图片描述](/img/e09483819498445ea89bd0ffabb19f9d.png)

---
# 参考
1.[https://ollama.com/](https://ollama.com/)
2.[https://lmstudio.ai/](https://lmstudio.ai/)
3.[https://manus.im/](https://manus.im/)
4.[https://www.trae.com.cn/?utm_campaign=r1](https://www.trae.com.cn/?utm_campaign=r1)
5.[https://datawhalechina.github.io/handy-ollama/#/](https://datawhalechina.github.io/handy-ollama/#/)
6.[https://github.com/ollama/ollama-python](https://github.com/ollama/ollama-python)
7.[https://github.com/coze-dev/coze-studio/](https://github.com/coze-dev/coze-studio/)

