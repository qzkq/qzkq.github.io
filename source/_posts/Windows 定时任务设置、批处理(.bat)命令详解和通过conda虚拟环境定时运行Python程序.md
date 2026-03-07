---
title: Windows 定时任务设置、批处理(.bat)命令详解和通过conda虚拟环境定时运行Python程序
date: 2026-01-03 08:18:00
tags:
  - "技术笔记"
categories:
  - "技术笔记"
---

﻿@[TOC](Windows 定时任务设置、批处理bat命令详解和通过conda虚拟环境定时运行Python程序)
# Windows 定时任务设置详解

## 方法一：通过图形界面创建任务

1. **打开任务计划程序**
   - 按 Win+R，输入 `taskschd.msc` 并回车
   - 或在控制面板中找到"管理工具" → "任务计划程序"

2. **创建基本任务**
   - 在右侧操作栏点击"创建基本任务"
   - 输入名称和描述，例如"生产报告生成任务"

3. **设置触发器**
   - 选择执行频率：每天、每周或每月
   - 设置具体时间（如每天凌晨1点）

4. **设置操作**
   - 选择"启动程序"
   - 在"程序或脚本"字段中浏览选择您的 `.bat` 文件
   - 在"起始于"字段中输入批处理文件所在目录的路径

5. **完成设置**
   - 查看摘要并完成创建

## 方法二：使用命令行创建（推荐用于自动化部署）

```cmd
schtasks /create /tn "生产报告生成任务" /tr "E:\path\to\run_report.bat" /sc daily /st 01:00 /ru SYSTEM /rl HIGHEST
```

### 参数详解：

- `/tn "任务名称"` - 指定任务的名称
- `/tr "任务运行的程序或命令"` - 指定要运行的程序或脚本的路径
- `/sc 日程计划` - 指定计划频率：
  - `daily` - 每天
  - `weekly` - 每周
  - `monthly` - 每月
  - `onstart` - 系统启动时
  - `onlogon` - 用户登录时
  - `onidle` - 系统空闲时
- `/st HH:MM` - 指定开始时间（24小时格式）
- `/ru 用户名` - 指定运行任务的用户账户（使用`SYSTEM`表示系统账户）
- `/rl 权限级别` - 指定权限级别：
  - `HIGHEST` - 最高权限
  - `LIMITED` - 标准权限

### 其他有用参数：

- `/mo 修饰符` - 指定任务 within 其计划类型的频率：
  - 对于每月任务：`/mo 1` 表示每1个月
  - 对于每周任务：`/mo 2` 表示每2周
- `/d 星期` - 指定一周中的某天或某些天（MON、TUE、WED等）
- `/ed MM/DD/YYYY` - 指定任务计划的结束日期
- `/sd MM/DD/YYYY` - 指定任务计划的开始日期
- `/it` - 仅在指定用户登录时才运行任务（与`/ru`一起使用）

## 示例命令

1. **每天凌晨1点运行**：

   ```cmd
   schtasks /create /tn "生产报告每日生成" /tr "E:\reports\run_report.bat" /sc daily /st 01:00
   ```

2. **每周一早上6点运行**：

   ```cmd
   schtasks /create /tn "生产报告每周生成" /tr "E:\reports\run_report.bat" /sc weekly /d MON /st 06:00
   ```

3. **每月第一天运行**：

   ```cmd
   schtasks /create /tn "生产报告每月生成" /tr "E:\reports\run_report.bat" /sc monthly /mo 1 /d 1 /st 02:00
   ```

## 管理现有任务

- 查看所有任务：`schtasks /query`
- 删除任务：`schtasks /delete /tn "任务名称"`
- 运行任务：`schtasks /run /tn "任务名称"`
- 结束任务：`schtasks /end /tn "任务名称"`

# Windows 批处理(.bat)命令详解

批处理文件是包含一系列 DOS 命令的文本文件，用于自动化执行任务。

## 基础命令

### 1. @echo

```bat
@echo off      # 关闭命令回显（不显示执行的命令本身）
@echo on       # 开启命令回显
echo Hello     # 输出文本到屏幕
echo.          # 输出空行
```

### 2. rem 和 ::

```bat
rem 这是注释    # 官方注释语句
:: 这也是注释   # 常用但非官方的注释方式
```

### 3. pause

```bat
pause          # 暂停执行，显示"请按任意键继续..."
```

### 4. title

```bat
title 我的批处理程序 # 设置命令窗口的标题
```

## 变量操作

### 5. set

```bat
set var=value      # 定义变量
set var=           # 清空变量
set /p var=请输入: # 接收用户输入
set /a result=1+1  # 数学计算
setlocal           # 开始局部环境变量
endlocal           # 结束局部环境变量
```

### 6. 特殊变量

```bat
%0 - 批处理文件本身
%1, %2, ... - 命令行参数
%* - 所有参数
%CD% - 当前目录
%DATE% - 当前日期
%TIME% - 当前时间
%RANDOM% - 随机数
%ERRORLEVEL% - 上条命令的退出代码
```

### 7. 参数扩展

```bat
%~dp0       # 当前批处理文件所在目录的完整路径
%~n0        # 当前批处理文件名（不含扩展名）
%~x0        # 当前批处理文件扩展名
```

## 流程控制

### 8. if 条件判断

```bat
if condition command

:: 比较字符串
if "str1"=="str2" command
if not "str1"=="str2" command

:: 检查文件存在
if exist filename command
if not exist filename command

:: 检查变量是否定义
if defined var command

:: 数值比较
if %n1% equ %n2% command  # 等于
if %n1% neq %n2% command  # 不等于
if %n1% lss %n2% command  # 小于
if %n1% leq %n2% command  # 小于等于
if %n1% gtr %n2% command  # 大于
if %n1% geq %n2% command  # 大于等于

:: 多行条件
if condition (
    command1
    command2
) else (
    command3
    command4
)
```

### 9. for 循环

```bat
:: 简单循环
for %%i in (1,2,3) do echo %%i

:: 文件循环
for %%i in (*.txt) do echo %%i

:: 数字范围循环
for /l %%i in (1,1,5) do echo %%i

:: 解析文本内容
for /f "delims= tokens=1,2" %%i in (file.txt) do echo %%i %%j

:: 解析命令输出
for /f "delims=" %%i in ('dir /b') do echo %%i
```

## 文件操作

### 10. 目录操作

```bat
cd \path\to\dir     # 切换目录
cd /d D:\path       # 切换驱动器并目录
dir                 # 列出文件
mkdir folder        # 创建目录
rmdir folder        # 删除目录
rmdir /s /q folder  # 强制删除目录（无确认）
```

### 11. 文件操作

```bat
copy file1 file2    # 复制文件
xcopy src dest      # 复制目录
move file1 file2    # 移动/重命名文件
del file.txt        # 删除文件
del /q *.tmp        # 安静模式删除
type file.txt       # 显示文件内容
```

### 12. 重定向

```bat
command > file.txt     # 输出重定向（覆盖）
command >> file.txt    # 输出重定向（追加）
command < input.txt    # 输入重定向
command 2> error.log   # 错误输出重定向
command > output.log 2>&1  # 标准输出和错误都重定向
```

## 函数和调用

### 13. call

```bat
call another.bat      # 调用另一个批处理文件
call :label args      # 调用标签作为函数
```

### 14. goto

```bat
goto label           # 跳转到标签
:label               # 定义标签
```

### 15. 函数示例

```bat
@echo off
call :myfunction "参数1" "参数2"
goto :eof

:myfunction
echo 第一个参数: %~1
echo 第二个参数: %~2
goto :eof
```

## 错误处理

### 16. errorlevel

```bat
command
if errorlevel 1 (
    echo 命令执行失败
) else (
    echo 命令执行成功
)
```

### 17. 强制出错退出

```bat
ver > nul        # 总是成功的命令
echo %errorlevel% # 显示错误级别
```

## 实用技巧

### 18. 延迟变量扩展

```bat
setlocal enabledelayedexpansion
set var=1
for /l %%i in (1,1,3) do (
    set /a var=var+1
    echo !var!     # 使用!而不是%获取实时值
)
```

### 19. 颜色设置

```bat
color 0A          # 背景黑(0)，文字绿(A)
```

### 20. 超时等待

```bat
timeout /t 5      # 等待5秒
timeout /t 5 /nobreak # 等待5秒，不可用Ctrl+C中断
```

## 调试技巧

1. **逐行调试**: 移除 `@echo off` 查看每条命令执行
2. **输出变量**: 使用 `echo 变量值: %var%` 调试
3. **暂停查看**: 在关键位置添加 `pause`
4. **日志记录**: 使用 `>> log.txt` 重定向输出到日志文件



# 通过conda虚拟环境定时运行Python程序

```bash
@echo off
setlocal enabledelayedexpansion

rem ==== Miniconda 配置 ====
rem 自动检测 Miniconda 安装路径
if exist "E:\Miniconda3" (
    set "CONDA_ROOT=E:\Miniconda3"
) else (
    echo 未找到 Miniconda 安装路径
    exit /b 1
)

set "ENV_NAME=production_report"
set "PYTHON_SCRIPT=%~dp0ollama_analysis.py"
set "LOG_DIR=%~dp0logs"
set "REPORT_DIR=%~dp0output"

rem ==== 日期处理 ====
for /f "tokens=1-3 delims=/ " %%a in ('date /t') do (
    set DATE_STR=%%c%%a%%b
)
set LOG_FILE=%LOG_DIR%\production_report_%DATE_STR%.log

rem ==== 确保目录存在 ====
if not exist "%LOG_DIR%" mkdir "%LOG_DIR%"
if not exist "%REPORT_DIR%" mkdir "%REPORT_DIR%"

rem ==== 激活 Miniconda 环境 ====
call "%CONDA_ROOT%\Scripts\activate.bat" %ENV_NAME%
if !errorlevel! neq 0 (
    echo [%time%] 激活 Miniconda 环境失败 >> "%LOG_FILE%"
    exit /b 1
)

rem ==== 运行 Python 脚本 ====
echo [%time%] Miniconda 环境激活成功 >> "%LOG_FILE%"
echo [%time%] 开始运行生产报告生成脚本 >> "%LOG_FILE%"

python "%PYTHON_SCRIPT%" >> "%LOG_FILE%" 2>&1
set EXIT_CODE=!errorlevel!

rem ==== 检查执行结果 ====
if !EXIT_CODE! equ 0 (
    echo [%time%] 报告生成成功 >> "%LOG_FILE%"
) else (
    echo [%time%] 脚本执行失败，错误码: !EXIT_CODE! >> "%LOG_FILE%"
)

endlocal
exit /b %EXIT_CODE%
```
## 示例说明
```bat
@echo off
```

- 关闭命令回显，执行时不显示批处理文件中的命令本身，只显示命令的输出结果

```bat
setlocal enabledelayedexpansion
```

- 启用延迟环境变量扩展，允许在代码块内使用 `!var!` 语法实时获取变量值（而不是预扩展的 `%var%`）

```bat
rem ==== Miniconda 配置 ====
```

- `rem` 表示注释/备注，不会执行

```bat
rem 自动检测 Miniconda 安装路径
```

- 注释行，说明下一行代码的作用

```bat
if exist "E:\Miniconda3" (
    set "CONDA_ROOT=E:\Miniconda3"
) else (
    echo 未找到 Miniconda 安装路径
    exit /b 1
)
```

- 检查指定的 Miniconda 安装路径是否存在
- 如果存在，设置 `CONDA_ROOT` 环境变量为该路径
- 如果不存在，显示错误信息并使用 `exit /b 1` 退出批处理并返回错误代码 1

```bat
set "ENV_NAME=production_report"
```

- 设置 Conda 环境名称为 `production_report`

```bat
set "PYTHON_SCRIPT=%~dp0ollama_analysis.py"
```

- `%~dp0` 表示当前批处理文件所在的目录路径
- 设置 Python 脚本路径为当前目录下的 `ollama_analysis.py`

```bat
set "LOG_DIR=%~dp0logs"
```

- 设置日志目录为当前目录下的 `logs` 文件夹

```bat
set "REPORT_DIR=%~dp0output"
```

- 设置报告输出目录为当前目录下的 `output` 文件夹

```bat
rem ==== 日期处理 ====
```

- 注释行，说明下一部分代码的作用

```bat
for /f "tokens=1-3 delims=/ " %%a in ('date /t') do (
    set DATE_STR=%%c%%a%%b
)
```

- 获取当前系统日期并重新格式化为 `年月日` 格式（如 20230915）
- `date /t` 获取当前日期
- `tokens=1-3` 将日期分割为三部分
- `delims=/ ` 使用斜杠和空格作为分隔符
- `%%c%%a%%b` 重新组合日期部分（年、月、日）

```bat
set LOG_FILE=%LOG_DIR%\production_report_%DATE_STR%.log
```

- 设置日志文件路径，包含日期字符串

```bat
rem ==== 确保目录存在 ====
```

- 注释行

```bat
if not exist "%LOG_DIR%" mkdir "%LOG_DIR%"
```

- 如果日志目录不存在，则创建该目录

```bat
if not exist "%REPORT_DIR%" mkdir "%REPORT_DIR%"
```

- 如果报告输出目录不存在，则创建该目录

```bat
rem ==== 激活 Miniconda 环境 ====
```

- 注释行

```bat
call "%CONDA_ROOT%\Scripts\activate.bat" %ENV_NAME%
```

- 调用 Miniconda 的激活脚本，激活指定的 Conda 环境
- `call` 用于调用另一个批处理文件并返回到当前批处理

```bat
if !errorlevel! neq 0 (
    echo [%time%] 激活 Miniconda 环境失败 >> "%LOG_FILE%"
    exit /b 1
)
```

- 检查上一条命令的退出代码（使用延迟扩展 `!errorlevel!`）
- 如果退出代码不等于 0（表示失败），则将错误信息追加到日志文件并退出

```bat
rem ==== 运行 Python 脚本 ====
```

- 注释行

```bat
echo [%time%] Miniconda 环境激活成功 >> "%LOG_FILE%"
```

- 将成功信息追加到日志文件，包含当前时间

```bat
echo [%time%] 开始运行生产报告生成脚本 >> "%LOG_FILE%"
```

- 将开始运行脚本的信息追加到日志文件

```bat
python "%PYTHON_SCRIPT%" >> "%LOG_FILE%" 2>&1
```

- 运行 Python 脚本
- `>> "%LOG_FILE%"` 将标准输出追加到日志文件
- `2>&1` 将标准错误重定向到标准输出（即错误信息也会写入日志文件）

```bat
set EXIT_CODE=!errorlevel!
```

- 将 Python 脚本的退出代码保存到变量 `EXIT_CODE` 中

```bat
rem ==== 检查执行结果 ====
```

- 注释行

```bat
if !EXIT_CODE! equ 0 (
    echo [%time%] 报告生成成功 >> "%LOG_FILE%"
) else (
    echo [%time%] 脚本执行失败，错误码: !EXIT_CODE! >> "%LOG_FILE%"
)
```

- 检查 Python 脚本的退出代码
- 如果为 0（成功），记录成功信息
- 如果不为 0（失败），记录失败信息和错误代码

```bat
endlocal
```

- 结束局部环境变量设置，恢复之前的变量状态

```bat
exit /b %EXIT_CODE%
```

- 退出批处理文件，并返回 Python 脚本的退出代码
- `/b` 参数表示只退出当前批处理文件，而不退出命令提示符窗口

![在这里插入图片描述](/img/b180ffdc6ab34742ad0d487a01211fa4.png)

