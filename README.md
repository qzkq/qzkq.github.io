# 基于Hexo的主题Fluid搭建Github博客

![GitHub repo size](https://img.shields.io/github/repo-size/qzkq/qzkq.github.io?style=for-the-badge)
![GitHub stars](https://img.shields.io/github/stars/qzkq/qzkq.github.io?style=for-the-badge)
![GitHub forks](https://img.shields.io/github/forks/qzkq/qzkq.github.io?style=for-the-badge)
![GitHub commit activity](https://img.shields.io/github/commit-activity/m/qzkq/qzkq.github.io?style=for-the-badge)

## 📒 简介

> :smiley: 此博客为自学数学建模和机器学习的分享博客，整理了许多笔记（**持续更新中......**）。
>
> :clap: 高考数学144；全国大学生数学竞赛一等奖（预赛）；华为杯中国研究生数学建模竞赛一等奖。
>
> :star: 基于Hexo下的fluid主题开发，以帮助更多数学建模和机器学习初学者，祝愿大家都能取得好成绩！如果觉得有用，请点个star 吧！感谢！！
>
> :triangular_flag_on_post: 同时，也欢迎大家PR，共同将这个项目壮大！

## 🤝 [在线阅读](https://qzkq.github.io/)

[公众号：数学建模与人工智能](https://mp.weixin.qq.com/s?__biz=MzI5MTY1MzU1Mg==&mid=2247487570&idx=1&sn=9800f79bf67614874faf4ace19d7081f&chksm=ec0c028ddb7b8b9b36862a0993f371307a8ec7f62df7138befab708723e5e7bd03a5c0544891&token=2142812089&lang=zh_CN#rd)

# 一、Github配置
## 1.安装Git
Git官网：[https://git-scm.com/downloads](https://git-scm.com/downloads)
安装步骤简单不再介绍，安装完成后通过`git --version`查看安装版本号。
## 2.部署本地Git与Github连接（SSH）
1.生成SSH KEY
```bash
ssh-keygen -t rsa -C *********@qq.com
```
注：邮箱填写github绑定的邮箱
2.查看.pub文件(.文件隐藏文件，使用ls -a显示隐藏文件)
```bash
cd ~/.ssh 切换目录到这个路径
vim id_rsa.pub 将这个文件的内容显示到终端上
```
也可以直接前往.shh文件所在的路径（前往~/.ssh这个路径），然后打开.pub这个文件，同样可以看到里面的内容。
3.将KEY添加到github的SSH Key里。
4.右键打开git bash，设置用户名和电子邮箱（注：这里的用户名和邮箱为你的github名称和绑定的邮箱）
```bash
git config --global user.name "****"
git config --global user.email "*********@qq.com"
```
5.通过github创建一个新的respository，名称为`github名字.github.io`（添加readme可以勾选上，后续可以介绍一下你的Bolg）
![在这里插入图片描述](https://img-blog.csdnimg.cn/d6388b4949b64d77946a37594b0937fb.png)
# 二、node.js安装和环境配置
## 1.安装node.js
node.js官网：[https://nodejs.org/zh-cn/](https://nodejs.org/zh-cn/)
exe安装直接Next即可（安装路径将默认C盘，建议更换到其他盘，这里我更换到D盘）
## 2.查看安装是否成功（版本号）

```bash
node -v
npm -v #nodejs安装默认附带安装npm
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ee89ace7c7a4e4ebabf291355eb966f.png)
## 3.配置环境变量
在nodejs安装目录下创建node_global，node_cache两个文件夹（全局安装的模块路径和缓存路径）

```bash
npm config set prefix "D:\nodejs\node_global"
npm config set cache "D:\nodejs\node_cache"
# 如显示权限不够，用管理员打开cmd
```
右键**此电脑**，点击**属性**，打开**高级系统设置**，**环境变量**，编辑系统变量，新建NODE_PATH变量，变量值为node_modules地址![在这里插入图片描述](https://img-blog.csdnimg.cn/7988105f2d034a40a458593f5b6263a2.png)
将用户变量下Path的`C:\Users\STAR\AppData\Roaming\npm`修改为`D:\nodejs\node_global`
![在这里插入图片描述](https://img-blog.csdnimg.cn/7359ade83e3643639e430f2b20b3336f.png)
防止npm下载过慢，使用国内淘宝镜像

```bash
npm config set registry https://registry.npm.taobao.org
```
# 三、下载Hexo并配置fluid主题
## 1.下载Hexo
通过cmd用npm安装hexo并初始化本地博客文件夹

```bash
npm install hexo-cli -g
hexo init ***.github.io  # 这里替换成你自己的，为后续更新到github上，使用github名字.github,io
cd ***.github.io  # 进入本地的博客文件夹
npm install
hexo server	# 打开本地服务器预览
```
之后通过浏览器查看[http://localhost:4000/](http://localhost:4000/)是否成功
注：可以通过`hexo -v`查看hexo安装版本
![在这里插入图片描述](https://img-blog.csdnimg.cn/026be8cbf4d740e6a9899f5f60be6830.png)
## 2.配置fluid主题
hexo主题官网： [https://hexo.io/themes/](https://hexo.io/themes/)，可以通过github搜索查看stars数大于3000的主题都有哪些，这里我们使用fluid（注：next主题使用的人很多，简约，教程很多这里不再介绍）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9de4476b7f2e4ba4a58a4de561f6a7a6.png)
### 1.安装fluid

```bash
npm install --save hexo-theme-fluid
```
通过npm安装的主题会放node_moduels里，然后在之前创建的博客目录下创建 _config.fluid.yml，将该主题下的 _config.yml 内容复制进去。
![在这里插入图片描述](https://img-blog.csdnimg.cn/b765a892c12044eea8b12d7934d4c96e.png)
### 2.配置fluid
修改博客目录中的 _config.yml，修改两个字段：

```bash
language: zh-CN  # 指定中文
theme: fluid  # 指定主题
```
其他配置可参考[Hexo Fluid 用户手册：配置指南](https://hexo.fluid-dev.com/docs/guide/#%E5%85%B3%E4%BA%8E%E6%8C%87%E5%8D%97)，介绍的很详细，`_config.fluid.yml`中也有很详细的注释。
![在这里插入图片描述](https://img-blog.csdnimg.cn/a22d99186ca94250a9873d8083732239.png)
### 3.更新部署博客页面
```bash
$ hexo clean  # 清空一下缓存，有时候博客页面显示不正常也可以试试这个命令行
$ hexo g  # hexo generate的简写，把刚刚做的改动生成更新一下
$ hexo server  # 在本地服务器看看博客有没有更新成NexT主题：https://localhost:4000
```
### 4.部署到github
修改博客根目录下的_config.yml文件中的deploy

```bash
deploy:
  type: git
  repo: git@github.com:qzkq/qzkq.github.io.git  # 这里我用的是ssh，也可以用https，可能会报错，设置token即可
  branch: main  # 注意自己创建的分支，我的是main，有可能是master
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/ab8f574b31b64485bfd3d26af63b0084.png)
安装hexo-deployer-git自动部署发布工具，将hexo 部署到 git page 的 deployer
```bash
npm install hexo-deployer-git --save
```

```bash
hexo d  # hexo deploy，如果本地服务器没问题就可以上传到github上
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/4c0e77efcbd3403b906236e6a0341e68.png)
更新博客之后，可以通过如下命令就行上传github

```bash
$ hexo clean	# 清空一下缓存，有时候博客页面显示不正常也可以试试这个命令行
$ hexo g	# 是hexo generate的简写，把刚刚做的改动生成更新一下
$ hexo server	# 在本地服务器看看博客有没有更新成NexT主题：https://localhost:4000
$ hexo d	# hexo deploy，如果本地服务器看了没问题就可以上传到github网站
```
# 四、美化
## 1.github-corners
在博客目录下的`node_modules\hexo-theme-fluid\layout\layout.ejs`的`<header>***</header>`中将从网站[https://tholman.com/github-corners/](https://tholman.com/github-corners/)中复制喜欢的颜色图标代码粘贴即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/1416119fd98644189481edc1d3e5ec1b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a413e4a425f444eac9cbe74b6fbe1b0.png)
## 2.背景图更换
推荐一个图片网址：[https://wallpaperhub.app/](https://wallpaperhub.app/)，可以选择尺寸下载。

# 展示：[https://qzkq.github.io/](https://qzkq.github.io/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fb584f03a56a4888be0c0d792b23e81a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b4b8e2852e6d4688885adbf7f77b40ba.png)
# 五、通过git提交代码到远程仓库
```c
重命名，用github代替git@github.com:****/******.git
git remote github git@github.com:****/******.git
```

1.查看有修改过的文件(**标红色的文件表示未提交到缓存区，绿色字表示已经添加到了缓存区**)
```c
git status
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/16114629edb74f13b7f1c0b2d26e432d.png)
2.把代码提交到缓存区,提交到缓存区后可以再次使用git status查看文件状态，如果所有文件都为绿色，证明已经添加到了缓存区

```c
git add .
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/49cee9fc20be4c34ae32f31b8650c7c1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6df9f1f34fb7400998b141a3b140c37b.png)
3.提交代码并备注

```c
git commit -m '备注'
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8bcac77394ad4612a5dbedc90c44ef92.png)
4.要保证你的代码是最新的，要拉取git上的代码

```c
git pull
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/d36ae953c7fe4436b51b80ead34e407c.png)
5.将代码推到git上（为了防止提交代码时出现错误，通常都会先提交到分支里面，确认不会出现问题以后再将代码合并到主分支）
**注：git push 用于从将本地的分支版本上传到远程并合并。**

```c
git push origin 分支名
```

> git push origin master 
> 将本地的 master 分支推送到 origin 主机的 master 分支

![在这里插入图片描述](https://img-blog.csdnimg.cn/78d7c3f469fd4dad8556d3c4265d6ed0.png)
main是用户qzkq创建的默认分支，master是用户QInzhengk创建的分支。
![在这里插入图片描述](https://img-blog.csdnimg.cn/266d138071284e80ac1a418d2138e8dc.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0931f8aadf7c498fa39ea4a5834fd36c.png)
