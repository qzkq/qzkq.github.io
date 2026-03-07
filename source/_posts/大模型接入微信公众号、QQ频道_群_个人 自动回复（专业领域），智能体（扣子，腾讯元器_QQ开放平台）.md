---
title: 大模型接入微信公众号、QQ频道_群_个人 自动回复（专业领域），智能体（扣子，腾讯元器_QQ开放平台）
date: 2026-01-04 08:07:00
tags:
  - "大模型"
  - "智能体"
  - "微信公众号"
  - "QQ"
  - "自动回复"
categories:
  - "大模型"
---

﻿@[TOC](大模型接入微信公众号、QQ频道/群/个人 自动回复（专业领域），智能体（扣子，腾讯元器/QQ开放平台）)
# 扣子
借助扣子提供的可视化设计与编排工具，你可以通过零代码或低代码的方式，快速搭建出基于大模型的各类 AI 项目，满足个性化需求、实现商业价值。
**智能体**：智能体是基于对话的 AI 项目，它通过对话方式接收用户的输入，由大模型自动调用插件或工作流等方式执行用户指定的业务流程，并生成最终的回复。智能客服、虚拟伴侣、个人助理、英语外教都是智能体的典型应用场景。
**应用**：应用是指利用大模型技术开发的应用程序。扣子中搭建的 AI 应用具备完整业务逻辑和可视化用户界面，是一个独立的 AI 项目。通过扣子开发的 AI 应用有明确的输入和输出，可以根据既定的业务逻辑和流程完成一系列简单或复杂的任务，例如 AI 搜索、翻译工具、饮食记录等。
## 智能体
### 步骤1：创建一个智能体（应用）
[https://www.coze.cn/docs/guides/quickstart](https://www.coze.cn/docs/guides/quickstart)
![在这里插入图片描述](/img/8a3cceaac8dc4706a0aa9ac424f5a446.png)

### 步骤2：编写提示词，为智能体添加技能，调试智能体（模型支持deepseek）
![在这里插入图片描述](/img/7466c2a51e704c8b8f360d41bcd661e7.png)
### 步骤3：发布智能体（可以接入微信公众号，api等）
![在这里插入图片描述](/img/49ab642dd1ad416086accff058ddc698.png)
## 应用
[开发一个 AI 翻译应用](https://www.coze.cn/open/docs/guides/app_quickstart)
### 业务逻辑
![在这里插入图片描述](/img/061daf7127574134a9c8ba2825a6749e.png)
### 用户界面
![在这里插入图片描述](/img/7d8eee48251646d9b23380e437cfa4fc.png)
### 发布（支持微信小程序和抖音小程序，需认证）
![在这里插入图片描述](/img/f4d21b03aebe4def998ffcb6225c14ac.png)
**模板**：[https://www.coze.cn/template](https://www.coze.cn/template)
# 腾讯元器(可以绑定QQ机器人)
[https://yuanqi.tencent.com/my-creation/agent](https://yuanqi.tencent.com/my-creation/agent)
## 1.智能体创建（支持deepseek等模型）
![在这里插入图片描述](/img/1545446cfddb447cb522f04240e8b89d.png)

## 2.发布和使用（可以接入微信公众号，api等）
![在这里插入图片描述](/img/58ae281f399047c481d3a1de6e863e50.png)
# QQ机器人（可以绑定腾讯元器，也可以自己购买云服务器开发）
## [python sdk](https://github.com/tencent-connect/botpy)
![在这里插入图片描述](/img/a5750e5cc9884e50962960615bb4e133.png)
## 使用腾讯云服务器开发QQ机器人（其余云服务器类似）
[腾讯云服务器](https://cloud.tencent.com/)
![在这里插入图片描述](/img/ebaa1e6556ad4697acb272de9c80a8f4.png)
### 首先购买一个腾讯云服务器，如果需求不多，最便宜的即可，学生有免费试用
[https://cloud.tencent.com/product/lighthouse](https://cloud.tencent.com/product/lighthouse)
我买的是最便宜的腾讯云轻量服务器一年99元，对于我自己的需求该配置已够用。
![在这里插入图片描述](/img/a77684ccd09540ce9ed2d77b0c0e6732.png)
你想要用它开发网站，可以选择WordPress和Halo建站。
![在这里插入图片描述](/img/9665a0c58fd240d5b1c9774d3862c188.png)
还可以购买域名，有些系统支持自动设置HTTPS，如不支持需手动配置。
![在这里插入图片描述](/img/1126d34ef42545519fae309fd92df365.png)
如需购买域名，一年几元即可拥有。
![在这里插入图片描述](/img/4d5bbaa52fbb4d289a4fc54b00a51587.png)
域名购买之后，可设置https，并且进行ICP备案，过程可直接在腾讯云操作，这里不再复述。
## Pycharm Pro远程连接云服务器
### Pycharm配置ssh
打开Pycharm->文件->设置->工具->SSH配置
![在这里插入图片描述](/img/c86d932671834f85bcd4afaa52b28e54.png)
主机和用户名密码写好之后点击 测试连接：出现成功连接即可（这里我之前用VSCode连接过，所以有`~/.shh/config`）。
![在这里插入图片描述](/img/927c2c4f7dd24b848b9bea8eeea2aec4.png)

### Pycharm部署远程服务器
打开 工具->部署->配置

 1. 连接：类型选择SFTP，SSH配置选择上步骤配好的。
![在这里插入图片描述](/img/a16ca01166664bc9865fcba12b5dd13f.png)
 2. 映射：设置本地路径和部署路径
![在这里插入图片描述](/img/a6c4490949714435a7d15e274022109e.png)
配置完成后点击 浏览远程主机 即可
![在这里插入图片描述](/img/8aebd6fc471241d4811afd0c70824bb9.png)
### Pycharm配置远程Python解释器
文件->项目（Python解释器）->添加解释器->基于SSH->SSH连接选择现有->点击下一步
![在这里插入图片描述](/img/de0bb16c26734df98a64e186fe856aae.png)
可以选择虚拟环境、系统解释器、conda环境：我下载的是minconda，所以选择的是conda环境；自动上传项目文件到服务器建议关闭，同步文件夹做一下更改。
![在这里插入图片描述](/img/e360ca7c4b1b486c90dc8e88cd85b694.png)
![在这里插入图片描述](/img/f0360d62a26649d9ac5b17caaa1baf01.png)

## VSCODE远程连接云服务器，方便代码开发
### 下载 RemoteSSH
[https://code.visualstudio.com/](https://code.visualstudio.com/)
在VSCODE中搜索并安装第一个, 下面两个会自动帮你装好。
![在这里插入图片描述](/img/0e0f375f62af4347831912f045b7c932.png)
### 配置 config
配置的格式为:

```bash
Host host1 # 主机的别名
    HostName  121.66.88.886  # 主机的 IP
    User root  ## 要通过 ssh 登陆的用户
```

 - Host: 显示的名字, 取个自己记得住的就行
 - HostName: 填服务器的 IP 地址
 - User: 要通过 ssh 登陆的用户。

### 连接
点击连接目标服务器, 输入密码;

如果你遇到报错, 请看报错自查部分, 可能会给你答案.

第一次登陆时, 服务器会下载 VScode Server, 这是正常的.

左下角出现服务器的别名, 说明登陆成功.

## 在VSCODE中编写python脚本
可参考：[https://cloud.tencent.com/lab/courseDetail/1012757351629305](https://cloud.tencent.com/lab/courseDetail/1012757351629305)

![在这里插入图片描述](/img/f8e8f53bc7024c06beea9f56f6508fd9.png)

### 设置使用范围和人员
![在这里插入图片描述](/img/da6135efdf6a4b1197b34fa8b787af87.png)

可添加自己，或者群聊和频道用来测试。并且ip白名单需要添加自己的云服务器公网ip。（回调地址为购买并备案icp成功的网址，测试不填也可。）
![在这里插入图片描述](/img/6b465f7e482d4256a219076392a9f9eb.png)
需在服务器安装
```bash
pip install qq-botpy
pip install pyyaml
```
**config.yaml**

```bash
token:
  appid: "123"
  token: "xxxx"
```

测试成功后，服务器会输出
![在这里插入图片描述](/img/22b489cfe3af46609178df3c81cb58fb.png)
### 示例代码

```python
# -*- coding: utf-8 -*-
import asyncio
import os
import random
import time
import datetime
import pandas as pd

from openai import OpenAI

import botpy
from botpy import logging
from botpy.ext.cog_yaml import read
from botpy.message import *

test_config = read(os.path.join(os.path.dirname(__file__), "config.yaml"))

_log = logging.get_logger()


class MyClient(botpy.Client):
    async def on_ready(self):
        _log.info(f"robot 「{self.robot.name}」 on_ready!")

    async def on_direct_message_create(self, message: DirectMessage):
        await self.api.post_dms(
            guild_id=message.guild_id,
            content=qzk(message),
            msg_id=message.id,
        )

    async def on_c2c_message_create(self, message: C2CMessage):
        await message._api.post_c2c_message(
            openid=message.author.user_openid,
            msg_type=0, msg_id=message.id,
            content=qzk(message)
        )

    async def on_group_at_message_create(self, message: GroupMessage):
        messageResult = await message._api.post_group_message(
            group_openid=message.group_openid,
              msg_type=0,
              msg_id=message.id,
              content=qzk(message))
        _log.info(messageResult)

    async def on_at_message_create(self, message: Message):
        _log.info(message.author.avatar)
        if "sleep" in message.content:
            await asyncio.sleep(10)
        _log.info(message.author.username)

        qin = qzk(message)

        await message.reply(content=f"{self.robot.name}: {qin}")


if __name__ == "__main__":
    # 通过预设置的类型，设置需要监听的事件通道
    # intents = botpy.Intents.none()
    # intents.public_guild_messages=True

    # 通过kwargs，设置需要监听的事件通道
    intents = botpy.Intents(public_guild_messages=True, public_messages=True, direct_message=True)
    client = MyClient(intents=intents)
    client.run(appid=test_config["appid"], secret=test_config["secret"])

```
**也可接入大模型api，让QQ机器人回答你问的问题**

```python
client = OpenAI(api_key="******", base_url="https://api.deepseek.com")
response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": "代码智能生成：经过海量优秀开源代码数据训练，可根据当前代码文件及跨文件的上下文，为你生成行级/函数级代码、单元测试、代码优化建议等。沉浸式编码心流，秒级生成速度，让你更专注在技术设计，高效完成编码工作。"
                                      "研发智能问答：基于海量研发文档、产品文档、通用研发知识、阿里云的云服务文档和 SDK/OpenAPI 文档等进行问答训练，为你答疑解惑，助你轻松解决研发问题。"
                                      "AI 程序员：具备多文件代码修改和工具使用的能力，可以与开发者协同完成编码任务，如需求实现、问题解决、单元测试用例生成、批量代码修改等。"},
        {"role": "user", "content": message.content.split(" ")[-1]},
    ],
    stream=False
)

qin = response.choices[0].message.content
```
python sdk中给了以下示例

```python
examples/
.
├── README.md
├── config.example.yaml          # 示例配置文件（需要修改为config.yaml）
├── demo_announce.py             # 机器人公告API使用示例
├── demo_api_permission.py       # 机器人授权查询API使用示例
├── demo_at_reply.py             # 机器人at被动回复async示例
├── demo_at_reply_ark.py         # 机器人at被动回复ark消息示例
├── demo_at_reply_embed.py       # 机器人at被动回复embed消息示例
├── demo_at_reply_command.py     # 机器人at被动使用Command指令装饰器回复消息示例
├── demo_at_reply_file_data.py   # 机器人at被动回复本地图片消息示例
├── demo_at_reply_keyboard.py    # 机器人at被动回复md带内嵌键盘的示例
├── demo_at_reply_markdown.py    # 机器人at被动回复md消息示例
├── demo_at_reply_reference.py   # 机器人at被动回复消息引用示例
├── demo_dms_reply.py            # 机器人私信被动回复示例
├── demo_get_reaction_users.py   # 机器人获取表情表态成员列表示例
├── demo_guild_member_event.py   # 机器人频道成员变化事件示例
├── demo_interaction.py          # 机器人互动事件示例（未启用）
├── demo_pins_message.py         # 机器人消息置顶示例
├── demo_recall.py               # 机器人消息撤回示例
├── demo_schedule.py             # 机器人日程相关示例
```
### QQ智能体
![在这里插入图片描述](/img/7ba94fcee32547938e15d2898dff9bdf.jpeg)

# 软链接
为 Python3 和 pip3 添加软链接可以通过 ln 命令来实现。以下是具体步骤：
1. 查找 Python3 和 pip3 的安装路径
首先，你需要知道 Python3 和 pip3 的实际安装路径。可以使用以下命令查找：

```bash
which python3
which pip3
```
这些命令会输出类似如下的路径：

```bash
Python3: /usr/local/bin/python3
pip3: /usr/local/bin/pip3
```

2. 创建软链接
假设你想将 Python3 和 pip3 的软链接创建到 /usr/bin/ 目录下（这是一个常见的系统路径），你可以使用以下命令：

创建 Python3 软链接

```bash
sudo ln -s $(which python3) /usr/bin/python3
```

创建 pip3 软链接

```bash
sudo ln -s $(which pip3) /usr/bin/pip3
```
3. 验证软链接是否成功创建
你可以通过以下命令验证软链接是否成功创建：

```bash
ls -l /usr/bin/python3
ls -l /usr/bin/pip3
```
你应该看到类似如下的输出，表明软链接已经成功创建：

```bash
lrwxrwxrwx 1 root root 22 Oct  1 12:34 /usr/bin/python3 -> /usr/local/bin/python3
lrwxrwxrwx 1 root root 22 Oct  1 12:34 /usr/bin/pip3 -> /usr/local/bin/pip3
```
4. 更新环境变量（如果需要）
如果你希望在终端中直接输入 python 或 pip 来调用 Python3 和 pip3，可以进一步创建软链接：
创建 python 和 pip 软链接指向 Python3 和 pip3

```bash
sudo ln -s $(which python3) /usr/bin/python
sudo ln -s $(which pip3) /usr/bin/pip
```
## 示例

```bash
ln -s /root/python37/bin/python3.7 /usr/bin/python3
ln -s /root/python37/bin/pip3 /usr/bin/pip3

注意： 这里的 /root/python37/ 是我的 Python3 安装路径，和之前下载的安装包放在同一个位置。运行前请确认一下你的安装路径是否和我一样。
```

## 注意事项

 - 权限问题：创建软链接时可能需要管理员权限，因此使用了 sudo。
 - 冲突问题：确保 /usr/bin/ 下没有同名文件或链接，否则可能会导致冲突。如果有同名文件或链接，建议先备份或删除旧的链接。

[发布到QQ智能体](https://docs.qq.com/aio/p/scxmsn78nzsuj64?p=GjUaQ9C1AMRploKJkUSW5pZ)
[发布到微信公众号（订阅号、服务号）](https://docs.qq.com/aio/p/scxmsn78nzsuj64?p=UaURIMeI5yybhR1GbWJuaLw)
# 大模型提示词示例
## 使用deepseek设置课堂随机点名，最后生成html，点击就能运行

```html
<!DOCTYPE html>

```
![在这里插入图片描述](/img/472e41a89b704682936a76d4e6090476.png)

# 参考
1.[https://www.coze.cn/](https://www.coze.cn/)
2.[https://yuanqi.tencent.com/agent-shop](https://yuanqi.tencent.com/agent-shop)
3.[https://q.qq.com/#/](https://q.qq.com/#/)
4.[https://ollama.com/search](https://ollama.com/search)
5.[https://github.com/datawhalechina/coze-ai-assistant](https://github.com/datawhalechina/coze-ai-assistant)
6.[https://chat.deepseek.com/](https://chat.deepseek.com/)
7.[https://bot.q.qq.com/wiki/#](https://bot.q.qq.com/wiki/#)


