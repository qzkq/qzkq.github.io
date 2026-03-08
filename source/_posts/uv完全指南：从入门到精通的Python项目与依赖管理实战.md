---
title: uv完全指南：从入门到精通的Python项目与依赖管理实战
date: 2026-03-07 09:00:00
tags:
  - "Python"
  - "uv"
  - "包管理"
  - "依赖管理"
  - "虚拟环境"
categories:
  - "技术笔记"
---

@[TOC](uv完全指南：从入门到精通的Python项目与依赖管理实战)
![在这里插入图片描述](/img/9fa87de9aafb41b3a9377e1b50fb44c8.png)

---
## 一、uv 是什么？—— 定位与核心功能

一个用 **Rust** 编写的、**极快**的 Python 包与项目工作流工具，集成了以下功能：
*   **虚拟环境管理** (`venv`)
*   **Python 包安装器** (`pip`, `pip-compile`)
*   **项目/依赖管理器** (`pipenv`, `poetry`)
*   **跨平台运行器** (`hatch`, `pdm run`, `tox`)

**核心理念**：从"全局/一体化的环境管理"转向 **"为每个项目创建独立、轻量、声明式配置的环境"**。

---

## 二、为什么需要 uv？—— 传统工具链的困境

在 uv 出现前，Python 开发者面临典型的"工具链碎片化"问题：

```text
开发一个项目需要组合使用：
├─ pyenv      → 管理 Python 版本
├─ venv       → 创建虚拟环境
├─ pip        → 安装依赖包
├─ pip-tools  → 生成 lock 文件
├─ pipx       → 安装全局工具
└─ poetry     → 项目管理（但速度慢）
```

**核心痛点**：

- ⏱️ **速度慢**：pip 安装大型项目（如 Django + 50 个依赖）需 10+ 秒，Conda 需 45+ 秒
- 💾 **磁盘浪费**：每个虚拟环境重复复制相同包，10 个项目用同一版本 numpy 占用 10 份空间
- 🔗 **环境不一致**：本地能跑，CI（**持续集成**）/CD（**持续交付/部署**） 报错，"依赖地狱"频发
- 🧩 **工具割裂**：需记忆多套命令，切换成本高

**uv 的破局之道**：
- ✅ **Rust 驱动**：零解释器开销 + 激进并行下载/解压
- ✅ **全局缓存 + 硬链接**：同一包只存一份，环境创建毫秒级
- ✅ **一体化设计**：一个工具覆盖从 Python 安装到项目发布的全流程

---

## 三、安装 uv

### Windows 安装

#### 方案一：一键安装（首选）
1.  **以管理员身份**打开 **Windows PowerShell**。
2.  执行命令：
    ```powershell
    powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
    ```
3.  **关闭并重新打开终端**，验证：
    ```bash
    uv --version
    ```

#### 方案二：使用 pip 安装
```bash
# 使用 pip 安装 uv
pip install uv

# 验证安装
uv --version
```
#### 方案三：手动安装
1.  **下载**：从 [GitHub Releases](https://github.com/astral-sh/uv/releases) 下载 `uv-x86_64-pc-windows-msvc.zip`。
2.  **解压**：解压到**无空格、无中文**的路径（如 `C:\tools\uv`）。
3.  **配置环境变量**：将解压路径添加到系统 `Path` 变量中。
4.  **验证**：重启终端，运行 `uv --version`。

#### 安装备用方案（当一键安装失败时，[安装指定版本](https://github.com/astral-sh/uv/releases/tag/0.10.0)）
```powershell
powershell -ExecutionPolicy Bypass -c "irm https://github.com/astral-sh/uv/releases/download/0.10.0/uv-installer.ps1 | iex"
```

### Linux 安装

#### 方案一：一键安装（推荐）
```bash
# 使用 curl
curl -LsSf https://astral.sh/uv/install.sh | sh

# 或者使用 wget
wget -qO- https://astral.sh/uv/install.sh | sh
```
安装完成后，重启终端或执行 `source ~/.local/bin/env`，然后运行 `uv --version` 验证。

#### 方案二：手动安装
1.  **下载**：从 [GitHub Releases](https://github.com/astral-sh/uv/releases) 下载对应的Linux版本。
2.  **解压并安装**：
    ```bash
    tar -xzf uv-x86_64-unknown-linux-gnu.tar.gz
    mv uv-x86_64-unknown-linux-gnu/uv ~/.local/bin/
    export PATH="$HOME/.local/bin:$PATH"
    ```
3.  **验证**：运行 `uv --version`。

### macOS 安装

#### 方案一：一键安装（推荐）
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
安装完成后，重启终端或执行 `source ~/.local/bin/env`，然后验证 `uv --version`。

#### 方案二：使用 Homebrew
```bash
brew install uv
```

#### 方案三：手动安装
步骤与Linux类似，从 [GitHub Releases](https://github.com/astral-sh/uv/releases) 下载对应架构的压缩包，解压后移动可执行文件到 `PATH` 路径。

---

## 四、核心工作流与命令

### 1. 项目初始化

运行 `uv init my_project` 会创建一个**简化、现代的项目结构**：
```
my_project/
├── .gitignore          # Git忽略文件
├── .python-version     # Python版本指定
├── main.py             # 主入口文件
├── pyproject.toml      # 项目配置文件
└── README.md           # 项目说明文档
```

### 1.1 .gitignore 文件详解

`.gitignore` 是 Git 版本控制系统的**忽略文件**，用于指定哪些文件或目录不应该被 Git 跟踪和提交到版本库。uv 自动生成的 `.gitignore` 文件包含了 Python 项目的最佳实践配置：

```gitignore
# Python-generated files
__pycache__/
*.py[oc]
build/
dist/
wheels/
*.egg-info/

# Virtual environments
.venv
```

#### 各部分详解：

##### 1. Python 生成的中间文件
- **`__pycache__/`**：Python 解释器缓存目录
  - Python 在导入模块时会将字节码缓存到 `__pycache__` 目录
  - 这些文件是机器相关的，不应提交到版本库
  
- **`*.py[oc]`**：Python 编译文件模式匹配
  - `*.pyc`：Python 字节码文件
  - `*.pyo`：优化后的字节码文件（Python 3.5+ 已弃用）
  - 这些文件会在运行时自动生成，不应提交

##### 2. 构建和分发产物
- **`build/`**：构建目录
  - 使用 `python setup.py build` 或 `uv build` 时生成的临时构建文件
  - 每次构建都会重新生成
  
- **`dist/`**：分发目录
  - 包含使用 `uv build` 生成的 `.whl` 和 `.tar.gz` 包
  - 这些是发布产物，应在 CI/CD 中生成，不应提交到源码库
  
- **`wheels/`**：wheel 缓存目录
  - 某些工具（如 pip）会缓存下载的 wheel 文件
  
- **`*.egg-info/`**：Egg 元信息目录
  - 旧式 Python 包的元数据目录（setuptools）
  - 现代项目应使用 `pyproject.toml` 替代

##### 3. 虚拟环境
- **`.venv`**：虚拟环境目录（推荐名称）
  - 使用 `uv venv` 创建的虚拟环境
  - **永远不要提交虚拟环境**到版本库，因为：
    1. 包含系统特定的二进制文件
    2. 占用大量空间（可能数百MB）
    3. 可以通过 `pyproject.toml` 和 `uv sync` 完全重现
    4. 可能包含敏感信息（如调试配置）

#### 为什么这些文件需要被忽略？

1. **可重现性**：项目应通过声明式配置（`pyproject.toml`）重现环境，而非提交具体文件
2. **清洁历史**：避免提交生成的中间文件，保持提交历史的清洁
3. **跨平台兼容**：某些文件是平台特定的（如 `.pyc` 文件格式可能因 Python 版本而异）
4. **安全性**：虚拟环境可能包含调试信息或临时密钥

#### 如何验证忽略规则？
```bash
# 查看哪些文件会被 Git 忽略
git status --ignored

# 检查特定文件是否被忽略
git check-ignore -v path/to/file
```

#### 自定义扩展
根据项目需要，可以在 `.gitignore` 中添加更多规则：

```gitignore
# IDE 配置文件（根据需要添加）
.vscode/
.idea/
*.swp
*.swo

# 环境变量文件（通常包含敏感信息）
.env
.env.local
.env.*.local

# 测试覆盖率报告
.coverage
htmlcov/

# 日志文件
*.log
logs/

# 操作系统文件
.DS_Store  # macOS
Thumbs.db  # Windows

# Jupyter 笔记本检查点
.ipynb_checkpoints/
```

### 2. 深入理解 `pyproject.toml`

`pyproject.toml` 是 **现代 Python 项目的核心配置文件**，它替代了传统的 `setup.py`、`requirements.txt`、`setup.cfg` 等文件，将所有项目元数据、依赖、构建配置统一在一个文件中。

#### `pyproject.toml` 示例与结构解析

```toml
[project]
name = "my-project"
version = "0.1.0"
description = "My awesome project"
requires-python = ">=3.11"
dependencies = [
    "requests>=2.31.0",
    "pandas>=2.0.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project.optional-dependencies]
dev = ["pytest", "black", "mypy"]
```

#### `[project]` 部分：项目基本信息
- **`name`**：项目名称，用于包索引（如 PyPI）
- **`version`**：项目版本，遵循语义化版本规范
- **`description`**：简短的项目描述
- **`readme`**：指向项目的 README 文件
- **`requires-python`**：指定项目支持的 Python 版本范围
- **`dependencies`**：项目运行所必需的依赖包列表

#### 项目元信息
```toml
authors = [{name = "Your Name", email = "your.email@example.com"}]
license = {text = "MIT"}
classifiers = [
    "Development Status :: 3 - Alpha",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3.13",
]
urls = {Homepage = "https://github.com/username/table-recognition"}
```

#### 命令行工具入口点
```toml
[project.scripts]
table-rec = "table_recognition.cli:main"
```
- 定义可以直接在终端中执行的命令
- 格式：`命令名 = "模块路径:函数名"`

#### 构建系统配置
```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```
- 当你执行 `uv build` 或 `pip install .` 时，会使用这里指定的构建系统

#### 可选依赖
```toml
[project.optional-dependencies]
dev = ["pytest>=7.4.0", "black>=23.0.0", "mypy>=1.5.0", "pre-commit>=3.0.0"]
test = ["pytest>=7.4.0", "pytest-cov>=4.0.0"]
docs = ["sphinx>=7.0.0", "sphinx-rtd-theme>=1.0.0"]
```
- 定义按功能分组的可选依赖
- 可以使用 `uv sync --group dev` 安装特定组的依赖

#### uv 工具特定配置
```toml
[tool.uv]
dev-dependencies = ["pytest", "black", "mypy"]
```
- `[tool.*]` 部分是给特定工具使用的配置
- `dev-dependencies`：指定开发依赖，`uv sync` 会自动安装这些包

### 3. `uv.lock` 文件详解

`uv.lock` 是 uv 用于**锁定依赖版本**的锁文件，确保在不同机器、不同时间安装的依赖完全一致。

```toml
version = 1
revision = 3
requires-python = ">=3.13"
resolution-markers = [
    "sys_platform == 'darwin'",
    "platform_machine == 'aarch64' and sys_platform == 'linux'",
    "(platform_machine != 'aarch64' and sys_platform == 'linux') or (sys_platform != 'darwin' and sys_platform != 'linux')",
]
```

#### 文件头部信息
- **`version = 1`**：锁文件格式版本
- **`revision = 3`**：锁文件修订号，每次更新依赖会增加
- **`requires-python = ">=3.13"`**：项目所需的最低 Python 版本
- **`resolution-markers`**：平台标记，用于跨平台依赖解析，如区分 macOS、Linux ARM、Linux x86 等

#### 包信息部分
每个包以 `[[package]]` 开头，包含以下字段：

```toml
[[package]]
name = "aiohttp"
version = "3.13.2"
source = { registry = "https://pypi.org/simple" }
dependencies = [
    { name = "aiohappyeyeballs" },
    { name = "aiosignal" },
    # ... 其他依赖
]
sdist = { 
    url = "https://files.pythonhosted.org/packages/.../aiohttp-3.13.2.tar.gz", 
    hash = "sha256:40176a52c186aefef6eb3cad2cdd30cd06e3afbe88fe8ab2af9c0b90f228daca", 
    size = 7837994, 
    upload-time = "2025-10-28T20:59:39.937Z" 
}
wheels = [
    { 
        url = "https://files.pythonhosted.org/packages/.../aiohttp-3.13.2-cp313-cp313-macosx_10_13_universal2.whl", 
        hash = "sha256:7519bdc7dfc1940d201651b52bf5e03f5503bda45ad6eacf64dda98be5b2b6be", 
        size = 732139, 
        upload-time = "2025-10-28T20:57:02.455Z" 
    },
    # ... 其他平台的 wheel 文件
]
```

#### 字段解析
- **`name`**：包名（如 `aiohttp`）
- **`version`**：锁定的确切版本（如 `3.13.2`）
- **`source`**：包来源，通常是 PyPI 注册表
- **`dependencies`**：该包的依赖列表
- **`sdist`**：源代码分发（`.tar.gz`）的详细信息：
  - `url`：下载地址
  - `hash`：SHA256 哈希值，用于校验文件完整性
  - `size`：文件大小（字节）
  - `upload-time`：上传时间戳
- **`wheels`**：预编译的 wheel 文件列表，为不同平台和 Python 版本提供：
  - 包含多种平台：macOS、Linux、Windows
  - 多种架构：x86_64、arm64、aarch64 等
  - 多种 Python 版本：cp313、cp314 等

#### 为什么需要如此详细的锁文件？
1. **确定性安装**：确保每次安装完全相同的文件
2. **跨平台支持**：为不同平台提供对应的预编译包
3. **完整性验证**：通过哈希值防止文件被篡改
4. **离线安装支持**：可基于锁文件离线构建环境

#### 如何生成和更新锁文件？
```bash
# 生成或更新锁文件
uv lock

# 同步依赖（会自动更新锁文件）
uv sync
```

### 3.1 `pyproject.toml` 与 `uv.lock` 的关系

#### `pyproject.toml`
- **声明性文件**：定义项目元数据和依赖范围
- **可手动编辑**：开发者可以修改版本范围
- **人类可读**：明确表达了项目的意图

#### `uv.lock`
- **锁文件**：记录当前环境下**确切**的依赖版本
- **自动生成**：由 `uv sync` 或 `uv lock` 自动创建/更新
- **确保一致性**：用于复现完全相同的环境
- **不应手动编辑**：由工具管理

### 4. 虚拟环境管理

```bash
# 创建虚拟环境
uv venv
uv venv --python 3.11  # 指定Python版本

# 激活环境（传统方式）
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
```

### 5. 依赖管理（现代方式，推荐）

依赖声明在 `pyproject.toml` 中。

| 命令                     | 说明                                        |
| ------------------------ | ------------------------------------------- |
| `uv add requests pandas` | 添加包并**自动更新** `pyproject.toml`       |
| `uv sync`                | 根据 `pyproject.toml` **安装/同步**所有依赖 |
| `uv lock`                | 生成或更新锁文件 `uv.lock`，确保环境可复现  |
| `uv remove requests`     | 移除包并更新 `pyproject.toml`               |

### 6. 依赖管理（传统方式，用于迁移）
```bash
# 像 pip 一样使用（不推荐长期使用）
uv pip install numpy
uv pip install -r requirements.txt
uv pip freeze > requirements.txt
```

### 7. 运行与工具
```bash
# 无需激活环境，直接运行
uv run python main.py

# 临时运行全局工具
uvx cowsay "Hello"
```

### 8. 安装 `.whl` 文件
```bash
uv pip install ./package.whl
uv pip install https://example.com/package.whl
```

---

## 五、在 PyCharm 中配置 uv

### 为新项目配置：
1.  在 `New Project` 窗口的 `Python Interpreter` 部分，直接选择 **`uv`**。
2.  PyCharm 将自动使用 `uv` 创建虚拟环境和项目文件。

### 为现有项目配置：
1.  `File` → `Settings` → `Project: <name>` → `Python Interpreter`。
2.  点击齿轮 ⚙ → `Add Interpreter` → `Add Local Interpreter`。
3.  在左侧选择 **`uv`** 作为解释器类型。
4.  选择 Python 版本或现有 `.venv` 路径，点击 `OK`。

**前提**：PyCharm 版本需为 **2025.2** 或更高，且系统已安装 `uv`。

---

## 六、从传统项目（比如Anaconda）迁移到 uv：Windows/Linux/macOS 完整指南

无论你是从 requirements.txt、Conda environment.yml 还是其他工具迁移，都可以按照以下步骤进行：

```bash
# 1. 进入项目目录
cd /path/to/your_project

# 2. 创建虚拟环境
uv venv

# 3. 激活环境（可选）
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate

# 4. 安装旧项目的依赖
uv pip install -r requirements.txt

# 5. 创建 pyproject.toml
uv init --bare

# 6. 迁移依赖到 pyproject.toml
# Windows（使用临时文件）：
uv pip freeze > temp.txt
uv add --requirements temp.txt
Remove-Item temp.txt

# Linux/macOS（命令替换）：
uv add $(uv pip freeze)

# 7. 同步依赖并生成锁文件
uv sync

# 8. 验证迁移是否成功
uv run python -c "import sys; print('Python版本:', sys.version)"
uv pip list
```

---

## 七、uv 与 Conda：深度对比与迁移指南

### 1. uv 与 Conda 的核心区别

| 维度         | **Conda**                               | **uv**                   |
| ------------ | --------------------------------------- | ------------------------ |
| **定位**     | 跨语言包管理器 + 环境管理器             | 纯 Python 项目工作流工具 |
| **包来源**   | Conda-forge、Anaconda 官方源（编译版）  | PyPI（Python 包索引）    |
| **包类型**   | Python + 非 Python（C 库、R、Julia 等） | **仅 Python 包**         |
| **环境隔离** | 完全隔离（包括系统库）                  | Python 级别隔离          |
| **性能**     | 较慢（依赖解析复杂）                    | **极快（10-100倍）**     |
| **磁盘占用** | 大（重复存储）                          | **小（硬链接缓存）**     |

### 2. 适用场景对比

#### 使用 Conda 的场景：
- ✅ **科学计算/数据科学**：需要 NumPy、SciPy 等与特定 BLAS 库链接的优化版本
- ✅ **跨语言项目**：Python + R + C++ 混合项目
- ✅ **系统级依赖**：需要特定版本的系统库（如 libc、OpenSSL）
- ✅ **Windows 开发**：需要编译工具的 C 扩展包

#### 使用 uv 的场景：
- ✅ **纯 Python 项目**：Web 开发、自动化脚本、工具开发
- ✅ **追求速度**：CI/CD 流水线、频繁创建/销毁环境
- ✅ **现代 Python 生态**：仅使用 PyPI 上的包
- ✅ **磁盘空间敏感**：个人笔记本、服务器资源有限

---

## 八、思维转换速查表

| 任务         | Anaconda / Conda 方式                 | **uv 方式（现代项目）**                          |
| ------------ | ------------------------------------- | ------------------------------------------------ |
| **创建环境** | `conda create -n my_env`              | `uv venv` （在项目目录生成 `.venv`）             |
| **声明依赖** | `environment.yml`                     | **`pyproject.toml`** （通过 `uv add` 编辑）      |
| **安装依赖** | `conda env update`                    | `uv sync` （根据 `pyproject.toml` 和 `uv.lock`） |
| **运行项目** | `conda activate` 然后 `python run.py` | `uv run python run.py` （无需手动激活）          |

---

## 九、总结：uv 与 Conda 的共存策略与未来展望

1. **纯 Python 项目** → **首选 uv**
   - Web 开发、API 服务、工具开发
   - CI/CD 流水线、Docker 容器
2. **科学计算/数据科学** → **评估后选择**
   - 如果 PyPI 包满足性能需求 → 使用 uv
   - 如果需要 MKL/OpenBLAS（**高性能数学计算库**） 优化 → 考虑 Conda 或混合方案
3. **跨语言项目** → **继续使用 Conda**
   - Python + R + C++ 混合项目
   - 需要特定版本系统库的项目
