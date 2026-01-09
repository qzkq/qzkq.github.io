# 基于 Hexo Fluid 的 GitHub 博客搭建指南

![GitHub repo size](https://img.shields.io/github/repo-size/qzkq/qzkq.github.io?style=for-the-badge)
![GitHub stars](https://img.shields.io/github/stars/qzkq/qzkq.github.io?style=for-the-badge)
![GitHub forks](https://img.shields.io/github/forks/qzkq/qzkq.github.io?style=for-the-badge)
![GitHub commit activity](https://img.shields.io/github/commit-activity/m/qzkq/qzkq.github.io?style=for-the-badge)

## 📒 简介

> :smiley: 此博客为自学数学建模和人工智能（机器学习、深度学习和大模型）的分享博客，整理了许多笔记（**持续更新中......**）。
>
> :clap: 高考数学144；全国大学生数学竞赛一等奖（预赛）；华为杯中国研究生数学建模竞赛一等奖。
>
> :star: 基于Hexo下的fluid主题开发，以帮助更多数学建模和机器学习初学者，祝愿大家都能取得好成绩！如果觉得有用，请点个star 吧！感谢！！
>
> :triangular_flag_on_post: 同时，也欢迎大家PR，共同将这个项目壮大！

## 🤝 在线阅读

**博客地址**: https://qzkq.github.io

**公众号**: [数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487570&idx=1&sn=9800f79bf67614874faf4ace19d7081f&chksm=ec0c028ddb7b8b9b36862a0993f371307a8ec7f62df7138befab708723e5e7bd03a5c0544891&token=2142812089&lang=zh_CN#rd)

---

## 一、GitHub 配置

### 1. 安装 Git

**Git 官网**: https://git-scm.com/downloads

安装步骤简单不再介绍，安装完成后通过以下命令查看安装版本号：

```bash
git --version
```

### 2. 部署本地 Git 与 GitHub 连接（SSH）

#### 步骤 1: 生成 SSH Key

```bash
ssh-keygen -t rsa -C "你的邮箱@qq.com"
```

**注意**: 邮箱填写 GitHub 绑定的邮箱。

#### 步骤 2: 查看公钥内容

```bash
cd ~/.ssh                          # 切换目录到这个路径
vim id_rsa.pub                      # 将这个文件的内容显示到终端上
```

也可以直接前往 `.ssh` 文件所在的路径（`~/.ssh`），然后打开 `id_rsa.pub` 这个文件，同样可以看到里面的内容。

#### 步骤 3: 将 Key 添加到 GitHub 的 SSH Key 里

1. 访问 https://github.com/settings/keys
2. 点击 `New SSH key`
3. 粘贴公钥内容
4. 保存

#### 步骤 4: 设置用户名和邮箱

```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的邮箱@qq.com"
```

#### 步骤 5: 创建 GitHub 仓库

通过 GitHub 创建一个新的 repository，名称为 `用户名.github.io`（添加 README 可以勾选上，后续可以介绍一下你的 Blog）

---

## 二、Node.js 安装和环境配置

### 1. 安装 Node.js

**Node.js 官网**: https://nodejs.org/zh-cn/

exe 安装直接 Next 即可（安装路径默认在 C 盘，建议更换到其他盘）。

### 2. 查看安装是否成功（版本号）

```bash
node -v
npm -v     # nodejs 安装默认附带安装 npm
```

### 3. 配置环境变量

在 Node.js 安装目录下创建 `node_global`、`node_cache` 两个文件夹（全局安装的模块路径和缓存路径）。

```bash
npm config set prefix "D:\nodejs\node_global"
npm config set cache "D:\nodejs\node_cache"
# 如显示权限不够，用管理员打开 cmd
```

配置系统环境变量：
- 右键**此电脑**，点击**属性**
- 打开**高级系统设置**，**环境变量**
- 编辑系统变量，新建 `NODE_PATH` 变量，变量值为 `node_modules` 地址
- 将用户变量下 Path 的 `C:\Users\用户名\AppData\Roaming\npm` 修改为 `D:\nodejs\node_global`

防止 npm 下载过慢，使用国内淘宝镜像：

```bash
npm config set registry https://registry.npm.taobao.org
```

---

## 三、下载 Hexo 并配置 Fluid 主题

### 1. 下载 Hexo

通过 cmd 用 npm 安装 hexo 并初始化本地博客文件夹：

```bash
npm install hexo-cli -g
hexo init 用户名.github.io          # 这里替换成你自己的，为后续更新到 github 上，使用用户名.github.io
cd 用户名.github.io                 # 进入本地的博客文件夹
npm install
hexo server                         # 打开本地服务器预览
```

之后通过浏览器查看 http://localhost:4000/ 是否成功。

**注**: 可以通过 `hexo -v` 查看 hexo 安装版本。

### 2. 配置 Fluid 主题

**Hexo 主题官网**: https://hexo.io/themes/

可以通过 GitHub 搜索查看 stars 数大于 3000 的主题都有哪些，这里我们使用 Fluid。

#### 安装 Fluid

```bash
npm install --save hexo-theme-fluid
```

通过 npm 安装的主题会放在 `node_modules` 里，然后在之前创建的博客目录下创建 `_config.fluid.yml`，将该主题下的 `_config.yml` 内容复制进去。

#### 配置 Fluid

修改博客目录中的 `_config.yml`，修改两个字段：

```yaml
language: zh-CN          # 指定中文
theme: fluid             # 指定主题
```

其他配置可参考 [Hexo Fluid 用户手册：配置指南](https://hexo.fluid-dev.com/docs/guide/)，介绍得很详细，`_config.fluid.yml` 中也有很详细的注释。

#### 更新部署博客页面

```bash
hexo clean               # 清空一下缓存，有时候博客页面显示不正常也可以试试这个命令
hexo g                   # hexo generate 的简写，把刚刚做的改动生成更新一下
hexo server              # 在本地服务器看看博客有没有更新成 Fluid 主题：https://localhost:4000
```

### 3. 部署到 GitHub

修改博客根目录下的 `_config.yml` 文件中的 deploy：

```yaml
deploy:
  type: git
  repo: git@github.com:你的用户名/你的用户名.github.io.git    # 这里我用的是 ssh，也可以用 https，可能会报错，设置 token 即可
  branch: main                                          # 注意自己创建的分支，我的是 main，有可能是 master
```

安装 hexo-deployer-git 自动部署发布工具，将 hexo 部署到 git page 的 deployer：

```bash
npm install hexo-deployer-git --save
```

部署到 GitHub：

```bash
hexo d                   # hexo deploy，如果本地服务器没问题就可以上传到 github 上
```

#### 更新博客常用命令

更新博客之后，可以通过如下命令上传到 GitHub：

```bash
hexo clean               # 清空一下缓存，有时候博客页面显示不正常也可以试试这个命令行
hexo g                   # 是 hexo generate 的简写，把刚刚做的改动生成更新一下
hexo server              # 在本地服务器看看博客有没有更新：https://localhost:4000
hexo d                   # hexo deploy，如果本地服务器看了没问题就可以上传到 github 网站
```

---

## 四、美化

### 1. GitHub Corners

在博客目录下的 `node_modules\hexo-theme-fluid\layout\layout.ejs` 的 `<header>***</header>` 中，将从网站 https://tholman.com/github-corners/ 中复制喜欢的颜色图标代码粘贴即可。

### 2. 背景图更换

推荐一个图片网址：https://wallpaperhub.app/，可以选择尺寸下载。

---

## 五、通过 Git 提交代码到远程仓库

### 配置远程仓库

```bash
# 重命名，用 github 代替 git@github.com:用户名/仓库名.git
git remote github git@github.com:你的用户名/你的仓库名.git
```

### 完整提交流程

#### 1. 查看有修改过的文件

```bash
git status
```

**说明**：标红色的文件表示未提交到缓存区，绿色字表示已经添加到了缓存区。

#### 2. 把代码提交到缓存区

```bash
git add .
```

提交到缓存区后可以再次使用 `git status` 查看文件状态，如果所有文件都为绿色，证明已经添加到了缓存区。

#### 3. 提交代码并备注

```bash
git commit -m '备注信息'
```

备注信息应该简洁明了地说明本次提交的内容，例如：
- "添加新文章：Hexo Fluid 博客搭建指南"
- "修改主题配置：更新导航栏菜单"
- "修复图片显示问题"

#### 4. 拉取 Git 上的代码

```bash
git pull
```

这一步可以确保你的代码是最新的，要拉取 git 上的代码。

#### 5. 将代码推到 Git 上

```bash
git push origin 分支名
```

**注**: `git push` 用于从将本地的分支版本上传到远程并合并。

```bash
git push origin main    # 将本地的 main 分支推送到 origin 主机的 main 分支
```

为了防止提交代码时出现错误，通常都会先提交到分支里面，确认不会出现问题以后再将代码合并到主分支。

**说明**：
- `main` 是新创建 GitHub 仓库的默认分支
- `master` 是旧版本的默认分支

### 分支管理最佳实践

```bash
# 1. 创建新分支用于开发
git checkout -b dev

# 2. 在分支上进行修改和提交
git add .
git commit -m '开发新功能'

# 3. 推送分支到远程仓库
git push github dev

# 4. 确认无误后，切换到主分支并合并
git checkout main
git merge dev

# 5. 推送主分支
git push github main
```

### .gitignore 配置

为了避免提交不必要的文件，建议在博客根目录创建 `.gitignore` 文件：

```gitignore
# Hexo 生成的静态文件
public/
db.json
*.log

# Node.js 依赖
node_modules/

# IDE 配置文件
.vscode/
.idea/
*.swp
*.swo
*~

# 系统文件
.DS_Store
Thumbs.db
```

---

## 六、博客更新步骤

### 1. 创建新文章

```bash
hexo new "文章标题"
```

这会在 `source/_posts/` 目录下生成一个 Markdown 文件。

### 2. 编辑文章

编辑生成的 Markdown 文件，设置文章的 Front-matter：

```markdown
---
title: 文章标题
date: 2026-01-09 12:00:00
categories: 分类名称
tags: [标签1, 标签2]
index_img: /img/cover.jpg      # 封面图
banner_img: /img/banner.jpg    # 文章页头图
---

文章正文内容...
```

### 3. 本地预览

```bash
hexo s
```

访问 http://localhost:4000 查看效果。

### 4. 生成静态文件

```bash
hexo clean
hexo g
```

### 5. 部署更新

```bash
hexo d
```

### 6. 查看更新

访问 `https://你的用户名.github.io` 查看更新后的博客。

---

## 七、常用命令汇总

### Hexo 命令

```bash
# 创建新文章
hexo new "文章标题"

# 创建新页面
hexo new page "页面名称"

# 启动本地服务器（端口 4000）
hexo server

# 清除缓存和静态文件
hexo clean

# 生成静态文件
hexo generate

# 部署到远程服务器
hexo deploy

# 组合命令：生成并部署
hexo clean && hexo g && hexo d
```

### Git 命令

```bash
# 初始化 Git 仓库
git init

# 添加远程仓库（重命名为 github）
git remote github git@github.com:你的用户名/你的仓库名.git

# 查看修改过的文件（红色表示未暂存，绿色表示已暂存）
git status

# 将代码提交到缓存区
git add .

# 提交代码并备注
git commit -m '备注信息'

# 拉取远程仓库最新代码
git pull

# 推送到远程仓库
git push github main

# 查看所有分支
git branch

# 创建新分支
git branch 分支名

# 切换分支
git checkout 分支名

# 创建并切换到新分支
git checkout -b 分支名

# 合并分支到当前分支
git merge 分支名

# 删除分支
git branch -d 分支名

# 查看提交历史
git log

# 查看远程仓库信息
git remote -v

# 暂存当前工作区的修改
git stash

# 恢复暂存的修改
git stash pop

# 丢弃本地修改
git checkout -- 文件名
```

---

## 八、项目结构说明

```
.
├── _config.yml           # Hexo 主配置文件
├── _config.fluid.yml     # Fluid 主题配置文件
├── package.json          # 项目依赖配置
├── scaffolds/            # 模板文件夹
├── source/               # 资源文件夹
│   ├── _posts/          # 文章源文件
│   ├── img/             # 图片资源
│   └── about/           # 关于页面
├── themes/              # 主题文件夹
├── public/              # 生成的静态文件（不要手动修改）
└── node_modules/        # Node.js 依赖（不要手动修改）
```

---

## 九、直接推送 public 文件夹到 GitHub Pages

如果需要直接将 Hexo 生成的静态文件推送到 GitHub Pages 仓库，可以使用以下命令：

```bash
# 1. 进入 public 文件夹（Hexo 生成的静态文件）
cd public

# 2. 初始化 git（如果还没初始化）
git init

# 3. 添加远程仓库（替换为你的仓库地址）
git remote add origin https://github.com/你的用户名/你的仓库名.git

# 4. 先拉取远程仓库最新代码（避免冲突）
git pull origin main --allow-unrelated-histories

# 5. 添加文件并提交
git add .
git commit -m "手动更新静态文件"

# 6. 推送到 GitHub Pages 分支（通常是 main、master 或 gh-pages）
git branch -M main
git push -u origin main

# 使用强制推送覆盖远程仓库
git push -u origin main --force
```

**注意**：
- 这种方式适用于不想使用 `hexo d` 命令自动部署的情况
- 强制推送（`--force`）会覆盖远程仓库，请谨慎使用
- 推送前先拉取（`git pull`）可以避免冲突，确保远程仓库最新
- `--allow-unrelated-histories` 参数用于合并两个不相关的历史记录
- 推送完成后记得回到项目根目录：`cd ..`

---

## 十、常见问题

### 1. 部署后页面 404

- 检查仓库名称是否正确（必须是 `用户名.github.io`）
- 检查 `_config.yml` 中的 `url` 配置
- 等待 GitHub Pages 构建完成（通常需要 1-3 分钟）

### 2. 图片无法显示

- 确保图片放在 `source/img/` 目录
- 使用相对路径引用：`/img/图片名.jpg`
- 图片名不要使用中文或特殊字符

### 3. 部署时提示权限错误

- 确保已配置 SSH 密钥
- 检查 `_config.yml` 中的仓库地址是否使用 SSH 格式
- 验证 GitHub 仓库权限

### 4. 本地预览正常，部署后样式丢失

- 运行 `hexo clean` 清除缓存
- 重新生成并部署：`hexo g && hexo d`
- 检查主题是否正确安装

### 5. 合并冲突

```bash
# 如果出现合并冲突，可以使用以下命令解决
git pull --rebase
# 手动解决冲突后
git add .
git rebase --continue
git push
```

### 6. 撤销提交

```bash
# 撤销最后一次提交（保留修改）
git reset --soft HEAD^

# 撤销最后一次提交（不保留修改）
git reset --hard HEAD^
```

---

## 十一、参考资源

- [Hexo 官方文档](https://hexo.io/zh-cn/docs/)
- [Fluid 主题文档](https://hexo.fluid-dev.com/docs/guide/)
- [Markdown 语法指南](https://markdown.com.cn/)
- [GitHub Pages 官方文档](https://docs.github.com/zh/pages)
- [GitHub Corners](https://tholman.com/github-corners/)
- [壁纸下载网站](https://wallpaperhub.app/)

---

## 十二、许可证

本博客内容采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议进行许可。

---

## 博客展示

[https://qzkq.github.io/](https://qzkq.github.io/)

---

**开始搭建你的博客吧！** 🎉

如果觉得这个项目对你有帮助，请点个 Star 支持一下！感谢！！
