---
title: 通过GitHub Actions给微信公众测试号和钉钉群定时推送消息（Python）
date: 2026-01-05 08:18:00
tags:
  - "GitHub"
  - "GitHub Actions"
  - "微信"
  - "钉钉"
  - "定时推送"
  - "Python"
categories:
  - "技术笔记"
---

﻿@[TOC](通过GitHub Actions给微信公众测试号和钉钉群定时推送消息（Python）)
## [https://github.com/QInzhengk/Math-Model-and-Machine-Learning](https://github.com/QInzhengk/Math-Model-and-Machine-Learning)
## 简介
GitHub Actions 是 GitHub 推出的持续集成服务。

> 在 GitHub Actions 的仓库中自动化、自定义和执行软件开发工作流程。 您可以发现、创建和共享操作以执行您喜欢的任何作业（包括 CI/CD），并将操作合并到完全自定义的工作流程中。

就是你可以给你的代码仓库部署一系列自动化脚本，在你进行了提交/合并分支等操作后，自动执行脚本。
## GitHub Actions 术语

 - workflow （工作流程）：持续集成一次运行的过程，就是一个 workflow。
 -  job （任务）：一个 workflow由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。
 -  step（步骤）：每个 job 由多个 step构成，一步步完成。 
 - action （动作）：每个 step 可以依次执行一个或多个命令（action）。

### workflow 文件
GitHub Actions 的配置文件叫做 workflow 文件，存放在代码仓库的.github/workflows目录。

workflow 文件采用 YAML 格式，文件名可以任意取，但是后缀名统一为.yml，默认为main.yml。一个库可以有多个 workflow 文件。GitHub 只要发现.github/workflows目录里面有.yml文件，就会自动运行该文件。
#### 1.name
workflow的名称。如果省略该字段，默认为当前workflow的文件名。
#### 2.on
触发workflow的条件，通常是某些事件，详细内容可以参照 [官方文档](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions) 。
例

```python
on: push
```

上面代码指定，push事件触发 workflow。

on字段也可以是事件的数组。


```python
on: [push, pull_request]
```

上面代码指定，push事件或pull_request事件都可以触发 workflow。

除了代码库事件，GitHub Actions 也支持外部事件触发，或者定时运行。

```python
on.<push|pull_request>.<tags|branches>
```

指定触发事件时，可以限定分支或标签。

```python
on:
  push:
    branches:    
      - master
```

上面代码指定，只有master分支发生push事件时，才会触发 workflow。

```python
on.schedule
```
后续定时推送微信公众测试号需要设置的；预定的工作流程在默认或基础分支的最新提交上运行。您可以运行预定工作流程的最短间隔是每 5 分钟一次。

在每天 5：30 和 17：30 UTC 触发工作流程（注意utc时间和中国时间差异）：

```python
on:
  schedule:
    # * is a special character in YAML so you have to quote this string
    - cron:  '30 5,17 * * *'
```
多个 事件可以触发单个工作流。你可以通过 上下文访问触发工作流的计划事件。此示例触发工作流在每周一至周四 5：30 UTC 运行，但在周一和周三跳过 步骤。

```python
on:
  schedule:
    - cron: '30 5 * * 1,3'
    - cron: '30 5 * * 2,4'

jobs:
  test_schedule:
    runs-on: ubuntu-latest
    steps:
      - name: Not on Monday or Wednesday
        if: github.event.schedule != '30 5 * * 1,3'
        run: echo "This step will be skipped on Monday and Wednesday"
      - name: Every time
        run: echo "This step will always run"
```

#### 3.jobs

```python
jobs.<job_id>.name
```

workflow 文件的主体是jobs字段，表示要执行的一项或多项任务。

jobs字段里面，需要写出每一项任务的job_id，具体名称自定义。job_id里面的name字段是任务的说明。

```python
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```

上面代码的jobs字段包含两项任务，job_id分别是my_first_job和my_second_job。

```python
jobs.<job_id>.needs
```

needs字段指定当前任务的依赖关系，即运行顺序。

```python
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

上面代码中，job1必须先于job2完成，而job3等待job1和job2的完成才能运行。因此，这个 workflow 的运行顺序依次为：job1、job2、job3。

```python
jobs.<job_id>.runs-on
```

runs-on字段指定运行所需要的虚拟机环境。它是必填字段。目前可用的虚拟机如下：

 - **ubuntu-lates**t，ubuntu-22.04或ubuntu-20.04
 -  **windows-latest**，windows-2022或windows-2019
 -  **macOS-latest**，macOS-12或macOS-11

指定虚拟机环境为ubuntu-20.04

```python
runs-on: ubuntu-20.04
```


```python
jobs.<job_id>.steps
```

steps字段指定每个 Job 的运行步骤，可以包含一个或多个步骤。每个步骤都可以指定以下三个字段。

 - jobs.<job_id>.steps.name：步骤名称。 
 - jobs.<job_id>.steps.run：该步骤运行的命令或者 action。 
 - jobs.<job_id>.steps.env：该步骤所需的环境变量。

### 几个完整的 workflow 文件的范例
#### 一
```python
#工作名字
name: qin
#
on:
  workflow_dispatch: 
  push:
  # 当对分支master进行push操作的时候，这个工作流就被触发了
    branches: [ master ]
  pull_request:
  #只运行特定分支master
    branches: [ master ]
  schedule:
  # 定时任务，在每天的24点 18点推送签到信息到邮箱
    - cron:  0 16 * * * 
  #watch:
  #    types: started   

jobs:
#将工作流程中运行的所有作业组合在一起
  kai:
  #定义名为 kai 的作业。 子键将定义作业的属性 
    runs-on: ubuntu-latest
    #将作业配置为在最新版本的 Ubuntu Linux 运行器上运行
    #if: github.event.repository.owner.id == github.event.sender.id
    # https://p3terx.com/archives/github-actions-manual-trigger.html
    
    steps:
    - uses: actions/checkout@v2
#uses 关键字指定此步骤将运行 actions/checkout 操作的 v3。 这是一个将存储
#库签出到运行器上的操作，允许您对代码（如生成和测试工具）运行脚本或其他操
#作。 每当工作流程将针对存储库的代码运行时，都应使用签出操作。
    
    - name: Set up Python 3.9
      uses: actions/setup-python@v2
      with:
        python-version: 3.9.1
    - name: requirements
      run: |
        python -m pip install --upgrade pip
        pip3 install -r requirements.txt
       # if [ -f requirements.txt ]; then pip install -r requirements.txt; fi 
    - name: Checkin
      run: |
        python3 ./test.py 
  env: 
  #设置secrets的环境变量
    COOKIE1: ${{ secrets.COOKIE1 }}
    COOKIE2: ${{ secrets.COOKIE2 }}
    #SMTP: ${{ secrets.SMTP }}
    MAIL1: ${{ secrets.MAIL1 }}
    MAIL2: ${{ secrets.MAIL2 }}
```
#### 二

```python
name: Schedule Worker
on:
  schedule:
    - cron: '40 3,9 * * *' #每日11点40,17点40，两个时间点执行任务
jobs:
  work:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: szenius/set-timezone@v1.0 # 设置执行环境的时区
        with:
          timezoneLinux: "Asia/Shanghai"
      - uses: actions/setup-python@v4 # 使用python装配器
        with:
          python-version: '3.10' # 指定python版本
          cache: 'poetry' # 设置缓存

      - run: poetry install --without dev # 安装
      - run: poetry run python .\bin.py # 执行
```
## 实例1、通过GitHub Actions给微信公众测试号定时推送消息（Python）
### 1、微信公众号步骤
打开 微信公众测试平台 ，通过微信扫一扫登录注册一个测试号。

**APP_ID**、**APP_SECRET**（复制自己的appID和appsecret）
![在这里插入图片描述](/img/7535c97904d5f51f15f49854c986be77.png)
**USER_ID**（复制关注测试号的用户微信号，可以给多个用户推送）
![在这里插入图片描述](/img/f856a9ab7678b56ba937fcccbe28cd9c.png)
自己新增测试模板（可根据自己喜好修改）
![在这里插入图片描述](/img/9912c03d219e3b18a2e5f6d9a2d900d2.png)

**TEMPLATE_ID**（模板ID）
![在这里插入图片描述](/img/2cce05c088eec23296f23c680c432867.png)
### 2、github步骤

新建或选取一个自己的公开库（也可以直接fork别人写好的），点击**Actions**，选择**New workflow**
![在这里插入图片描述](/img/36037309c0985d1c9fd7a5e6689bf421.png)
选择**set up a workflow yourself**，也可以搜索现成的wordflow
![在这里插入图片描述](/img/ab4dceef7506dd13a077aa53a55e1e47.png)
输入yml文件内容（可先什么也不写，后面会给出具体代码），选择**Start commit**
![在这里插入图片描述](/img/142cd3d6de6de250e0ba615373c400f1.png)
新建mian.py（可以在本地调试）和requirements.txt文件，可把python所需库版本放入（可先什么也不写，后面会给出具体代码）

之后点击**Settings**中的**Secrets**添加
![在这里插入图片描述](/img/0e47fbed916684a39364a5f62d80a5a7.png)
添加完成后（之前已经给出）：注意城市为北京，秦皇岛......；纪念日和生日格式需与main.py中一致。
![在这里插入图片描述](/img/f82ab744accd9576358503d73f25731c.png)
最后，点击**Acrions**里的**morning**，选择**Run workflow**，如果运行成功，则显示绿色对钩，并且用户会收到推送的消息。如运行失败，可以点击查看运行失败的错误信息是什么。
![在这里插入图片描述](/img/bdd8564011e2f34b4bf94b1fb2ae4c16.png)
代码地址：[https://github.com/QInzhengk/the_milky_way](https://github.com/QInzhengk/the_milky_way)
![在这里插入图片描述](/img/5f1f5941e14b1dfd353812d89330b696.png)

## 实例2、通过GitHub Actions给钉钉群聊机器人定时推送消息（Python）
### 1、钉钉步骤
打开钉钉，点击+发起群聊（如你有公司，需要有两个不是公司的好友才能创建普通群），创建完成后，打开群聊中的设置，智能群助手。
进入到机器人管理页面，点击添加机器人，进入机器人选择页面，这里选择自定义机器人。
![在这里插入图片描述](/img/51250e13138008fc932f3ce267a1cfe6.png)
需要给机器人修改头像和名称，在安全设置里面，建议最好把自定义关键字也勾选上，比如我这里设置的是：早上好，然后其他的可以默认，点击完成后在新的页面有一个webhook（Webhook不要泄露在网上）
![在这里插入图片描述](/img/0bf173237f6e4ec084edec8425c1ffad.png)
获取到Webhook地址后，用户可以向这个地址发起HTTP POST 请求，即可实现给该钉钉群发送消息。

钉钉群聊机器人最新规定：

> 发起POST请求时，必须将字符集编码设置成UTF-8。
> 每个机器人每分钟最多发送20条。消息发送太频繁会严重影响群成员的使用体验，大量发消息的场景 (譬如系统监控报警)可以将这些信息进行整合，通过markdown消息以摘要的形式发送到群里。

目前支持发送的消息有5种，分别是：文本 (text)、链接 (link)、markdown、ActionCard、FeedCard。个人使用中比较常用的有两种：分别是文本和链接，企业使用的时候，对于ActionCard类型的也比较常用。

具体需要根据自己的场景进行选择，以便能达到最好的展示样式。

自定义机器人发送消息时，可以通过手机号码指定“被@人列表”。在“被@人列表”里面的人员收到该消息时，会有@消息提醒。免打扰会话仍然通知提醒，首屏出现“有人@你”

#### 文本TEXT

文本型的消息类型，具体代码如下：

```
{
    "at": {
        "atMobiles":[
            "180xxxxxx"
        ],
        "atUserIds":[
            "user123"
        ],
        "isAtAll": false
    },
    "text": {
        "content":"测试"
    },
    "msgtype":"text"
}

```

上述中涉及的参数类型分别如下：

| **参数**  | **参数类型** | **是否必填** | **说明**                                                     |
| --------- | ------------ | ------------ | ------------------------------------------------------------ |
| msgtype   | String       | 是           | 消息类型，此时固定为：text。                                 |
| content   | String       | 是           | 消息内容。                                                   |
| atMobiles | Array        | 否           | 被@人的手机号。**注意** 在content里添加@人的手机号，且只有在群内的成员才可被@，非群内成员手机号会被脱敏。 |
| atUserIds | Array        | 否           | 被@人的用户userid。**注意** 在content里添加@人的userid。     |
| isAtAll   | Boolean      | 否           | 是否@所有人。                                                |

#### 链接LINK

链接型的消息类型，具体代码如下：

```
{
    "msgtype": "link", 
    "link": {
        "text": "测试", 
        "title": "测试", 
        "picUrl": "", 
        "messageUrl": "https://www.dingtalk.com/s?__biz=MzA4NjMwMTA2Ng==&mid=2650316842&idx=1&sn=60da3ea2b29f1dcc43a7c8e4a7c97a16&scene=2&srcid=09189AnRJEdIiWVaKltFzNTw&from=timeline&isappinstalled=0&key=&ascene=2&uin=&devicetype=android-23&version=26031933&nettype=WIFI"
    }
}
```

上述中涉及的参数类型分别如下：

| **参数**   | **参数类型** | 是否必填 | **说明**                                                     |
| ---------- | ------------ | -------- | ------------------------------------------------------------ |
| msgtype    | String       | 是       | 消息类型，此时固定为：link。                                 |
| title      | String       | 是       | 消息标题。                                                   |
| text       | String       | 是       | 消息内容。如果太长只会部分展示。                             |
| messageUrl | String       | 是       | 点击消息跳转的URL，打开方式如下：移动端，在钉钉客户端内打开PC端默认侧边栏打开希望在外部浏览器打开，请参考[消息链接说明](https://open.dingtalk.com/document/app/message-link-description#section-7w8-4c2-9az) |
| picUrl     | String       | 否       | 图片URL。                                                    |

#### markdown类型

markdown的消息类型，具体代码如下：

```json
{
     "msgtype": "markdown",
     "markdown": {
         "title":"测试",
         "text": "#### 杭州天气 @150XXXXXXXX \n > 9度，西北风1级，空气良89，相对温度73%\n > ![screenshot](/img/TB1NwmBEL9TBuNjy1zbXXXpepXa-2400-1218.png)\n > ###### 10点20分发布 [天气](https://www.dingtalk.com) \n"
     },
      "at": {
          "atMobiles": [
              "188XXXXXXXX"
          ],
          "atUserIds": [
              "user123"
          ],
          "isAtAll": false
      }
 }
```

上述中涉及的参数类型分别如下：

| **参数**  | **类型** | 是否必填 | **说明**                                                     |
| --------- | -------- | -------- | ------------------------------------------------------------ |
| msgtype   | String   | 是       | 消息类型，此时固定为：markdown。                             |
| title     | String   | 是       | 首屏会话透出的展示内容。                                     |
| text      | String   | 是       | markdown格式的消息。                                         |
| atMobiles | Array    | 否       | 被@人的手机号。**注意** 在text内容里要有@人的手机号，只有在群内的成员才可被@，非群内成员手机号会被脱敏。 |
| atUserIds | Array    | 否       | 被@人的用户userid。**注意** 在content里添加@人的userid。     |
| isAtAll   | Boolean  | 否       | 是否@所有人。                                                |



#### 整体跳转ActionCard类型

整体跳转ActionCard的消息类型，具体代码如下：

```
{
    "actionCard": {
        "title": "测试", 
        "text": "测试", 
        "btnOrientation": "0", 
        "singleTitle" : "测试",
        "singleURL" : "https://www.dingtalk.com/"
    }, 
    "msgtype": "actionCard"
}
```

上述中涉及的参数类型分别如下：

| **参数**       | **类型** | **是否必填** | **说明**                                                     |
| -------------- | -------- | ------------ | ------------------------------------------------------------ |
| msgtype        | String   | 是           | 消息类型，此时固定为：actionCard。                           |
| title          | String   | 是           | 首屏会话透出的展示内容。                                     |
| text           | String   | 是           | markdown格式的消息。                                         |
| singleTitle    | String   | 是           | 单个按钮的标题。**注意** 设置此项和singleURL后，btns无效。   |
| singleURL      | String   | 是           | 点击消息跳转的URL，打开方式如下：移动端，在钉钉客户端内打开PC端默认侧边栏打开希望在外部浏览器打开，请参考[消息链接说明](https://open.dingtalk.com/document/app/message-link-description#section-7w8-4c2-9az) |
| btnOrientation | String   | 否           | 0：按钮竖直排列1：按钮横向排列                               |


#### 独立跳转ActionCard类型

独立跳转ActionCard的消息类型，具体代码如下：

```
{
    "msgtype": "actionCard",
    "actionCard": {
        "title": "测试", 
        "text": "测试", 
        "btnOrientation": "0", 
        "btns": [
            {
                "title": "内容不错", 
                "actionURL": "https://www.dingtalk.com/"
            }, 
            {
                "title": "不感兴趣", 
                "actionURL": "https://www.dingtalk.com/"
            }
        ]
    }
}
```

上述中涉及的参数类型分别如下：

| **参数**       | **类型** | 是否必填 | 说明                                                         |
| -------------- | -------- | -------- | ------------------------------------------------------------ |
| msgtype        | String   | 是       | 此消息类型为固定actionCard。                                 |
| title          | String   | 是       | 首屏会话透出的展示内容。                                     |
| text           | String   | 是       | markdown格式的消息。                                         |
| btns           | Array    | 是       | 按钮。                                                       |
| title          | String   | 是       | 按钮标题。                                                   |
| actionURL      | String   | 是       | 点击按钮触发的URL，打开方式如下：移动端，在钉钉客户端内打开PC端默认侧边栏打开希望在外部浏览器打开，请参考[消息链接说明](https://open.dingtalk.com/document/app/message-link-description#section-7w8-4c2-9az) |
| btnOrientation | String   | 否       | 0：按钮竖直排列1：按钮横向排列                               |

#### FeedCard类型

FeedCard的消息类型，具体代码如下：

```
{
    "msgtype":"feedCard",
    "feedCard": {
        "links": [
            {
                "title": "测试1", 
                "messageURL": "https://www.dingtalk.com/", 
                "picURL": "https://img.alicdn.com/tfs/TB1NwmBEL9TBuNjy1zbXXXpepXa-2400-1218.png"
            },
            {
                "title": "测试2", 
                "messageURL": "https://www.dingtalk.com/", 
                "picURL": "https://img.alicdn.com/tfs/TB1NwmBEL9TBuNjy1zbXXXpepXa-2400-1218.png"
            }
        ]
    }
}
```

上述中涉及的参数类型分别如下：

| **参数**   | **类型** | 是否必填 | **说明**                                                     |
| ---------- | -------- | -------- | ------------------------------------------------------------ |
| msgtype    | String   | 是       | 此消息类型为固定feedCard。                                   |
| title      | String   | 是       | 单条信息文本。                                               |
| messageURL | String   | 是       | 点击单条信息到跳转链接。**说明** PC端跳转目标页面的方式，参考[消息链接在PC端侧边栏或者外部浏览器打开](https://open.dingtalk.com/document/app/message-link-description#section-7w8-4c2-9az)。 |
| picURL     | String   | 是       | 单条信息后面图片的URL。                                      |


### 2、Github步骤
这里和第一个实例类似，不再重复介绍。
代码地址：[https://github.com/QInzhengk/galaxy](https://github.com/QInzhengk/galaxy)
![在这里插入图片描述](/img/c6944006870923c037481ea22cde1a7e.png)
## 参考链接
1.[https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)
2.[https://docs.github.com/cn/actions/using-workflows/workflow-syntax-for-github-actions](https://docs.github.com/cn/actions/using-workflows/workflow-syntax-for-github-actions)
3.[https://github.com/13812851221/-rxrw-daily_morning](https://github.com/13812851221/-rxrw-daily_morning)
4.[https://blog.csdn.net/qq_45832050/article/details/126789897](https://blog.csdn.net/qq_45832050/article/details/126789897)
5.[https://blog.csdn.net/qq_45832050/article/details/122755904](https://blog.csdn.net/qq_45832050/article/details/122755904)
6.[https://github.com/datawhalechina/office-automation/tree/main/Task05-Python%E6%93%8D%E4%BD%9C%E9%92%89%E9%92%89%E8%87%AA%E5%8A%A8%E5%8C%96](https://github.com/datawhalechina/office-automation/tree/main/Task05-Python%E6%93%8D%E4%BD%9C%E9%92%89%E9%92%89%E8%87%AA%E5%8A%A8%E5%8C%96)
