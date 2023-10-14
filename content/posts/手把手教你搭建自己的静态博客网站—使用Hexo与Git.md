---
title: 手把手教你搭建自己的静态博客网站—使用Hexo与Git
date: 2022-07-10 10:55:19
tags:
    - 教程
    - 开发
    - Git
categories:
    - 建站
---

全文字数：3354
阅读时长：15min
文章类型：教程



### 介绍

---

这可能是你见过**最清楚的**，使用 **GitHub** 或 **Gitee** 与 **Hexo**搭建个人博客网站的教程，即使是小白，看不太懂某些概念，按照文中的方法流程操作，也可以完成网站的搭建。此篇教程的目的是搭建完成自己的博客并让它可以运行，更高级的定制操作将在后面介绍

编写时间为2022/7/10，教程中所用工具与网站以当前为准。



#### 基础

首先介绍一下这几个概念。

##### 静态网站

按照网站的组成和架构，可以将网站分为 静态网站和动态网站。它的**静态**体现在，用户访问的是完全固定的代码，除非网站的开发者做出一些更改，不然用户接收到的内容**永远不会改变**。二者主要有这样几个区别：

- 源代码。静态网站代码均是由 **HTML/CSS** 等静态标记语言组成的。而动态网站还会使用到 **JavaScript**，**PHP** 等语言。
- 组成。静态网站只有**前端页面**，也即用户能看到的部分。动态网站具有**数据库**（最主要的区别之一），**用户交互功能**等。
- 托管方式。访问静态网站一般只需访问其 **index.html** 以及与其相关的一系列文件，因此托管静态网站只需要一个能让用户直接访问到**这些**（百度网盘这些单文件访问不行，除非你只有一个index.html）文件的“东西”，甚至可以是经过内网穿透和映射的NAS。而动态网站则需要一个完整的服务器，以实现动态交互功能。
- 功能。静态网站的主要功能是，提供一些可以让用户看到的信息。动态网站则具有，与用户交换数据，动态访问与数据安全性等。

##### GitHub/Gitee

GitHub与Gitee均是基于 **git** 的代码托管平台（GitHub是全球使用最多的开源平台）。利用它们提供的  **Pages功能**，可以满足我们托管静态网站的需求，最重要的是，它们是**免费**的！

利用它们搭建静态网站需要频繁使用到git，不会使用的读者有必要了解一下git，参照以下链接：

[Git教程——菜鸟教程](https://www.runoob.com/git/git-tutorial.html)

由于GitHub目前的访问不太稳定，有时甚至无法访问，因此本教程还会介绍如何使用Gitee搭建博客。

##### Hexo

Hexo是一款简洁、高效的博客框架，使用Node.js 编写。目前多数托管在GitHub上的博客均使用Hexo。

Hexo使用**Markdown** 渲染引擎解析文章。**用户使用Markdown书写文章内容，Hexo 将其渲染成网页。**Hexo 有很多主题供你选择。

Markdown是一种轻量级的标记语言，或者说是排版语言，人们可以使用Markdown**快速编写具有一定排版的文章**，您现在正在阅读的这篇文章就是由Markdown写出的。Markdown的使用类似于Word，但它更能让你专注于内容写作。Markdown的学习较简单，不了解的读者有必要了解一下，参照以下链接：

[Markdown教程——菜鸟教程](https://www.runoob.com/markdown/md-tutorial.html)



#### 原理

用户访问一个网站，其大致流程是：

1. 在浏览器**输入域名**，按下回车 。
2. 经过一系列的**DNS信息查询**步骤，得到服务器的**IP地址。**
3. 通过得到的IP与端口，经过三次握手，与服务器建立**TCP连接**（不出意外的话，QUIC将在未来取代TCP）。
4. 通过**HTTP/HTTPS协议**请求网页内容，若通过，服务器便回复“OK”并下发网页内容。
5. 浏览器得到HTML内容，将其**解析渲染**，成为用户看到的网页。

我们使用GitHub或Gitee创建静态网页并没有涉及到服务器的搭建和处理，我们只需要关心我们网站内容的建设，这是一个**纯前端**的工作，而与客户端沟通等工作便交予服务器。等等，**我们的服务器呢？**

在这里，GitHub和Gitee的Pages服务便解决了我们的服务器需要的工作。当用户访问特定的域名时，经过DNS服务器获取IP后指向GitHub/Gitee的Pages服务器，并与之建立连接，当客户端发送请求时，服务器将找到在万千数据中相对应的，然后返回我们的网页给用户。**那我们的网页要放在哪里呢？**

我们只需要在GitHub/Gitee上创建一个**仓库**，将我们的网页代码放在上面就可以了（首页还是永远的index.html），我们在里面写什么，看到的网页就是什么。由于GitHub/Gitee Pages的服务基于我们创建的代码仓库，**因此我们只能在上面创建静态网站。**

也就是说，在GitHub/Gitee上做一些配置，就相当于我们把服务器创建好了，后面怎么创建网页内容完全是我们的自由。那么**Hexo是干什么的？**

Hexo如前面所说，就是一款博客框架，它的作用是帮我们创建出比较精美的博客网页，我们不需要再去学习HTML5之类的新的语言，把它交给前端，我们再去做属于自己的工作（如果你是一名前端工程师，完全可以尝试自己写出一个漂亮的网页）。

我们总体的**流程**就是：

1. 使用Hexo或自己**编写**，在电脑本地**生成**静态网站文件。
2. 使用git或Hexo deploy（基于git），将文件**同步**到我们的代码仓库。（你甚至可以**直接上传**到仓库，只要不嫌麻烦）
3. 在服务配置好的情况下，我们的网站便可以**访问**了。



#### 准备工作

这里对使用本教程的方法创建网站所需的内容列一清单，工具的Windows版本**官方**下载链接在下，Linux可使用apt自行安装。

**基础知识**：

- Git的使用（用于向GitHub或Gitee提交文件）[Git教程——菜鸟教程](https://www.runoob.com/git/git-tutorial.html)
- Markdown（学习简单，用于编写文章和内容）[Markdown教程——菜鸟教程](https://www.runoob.com/markdown/md-tutorial.html)

**需要工具**：

- 一台电脑

- Git（安装适用于自己系统的版本）[Git——下载](https://git-scm.com/downloads)
- Node.js （用于安装和使用 Hexo）[Node.js——下载](https://nodejs.org/en/download/)
- Hexo （博客搭建核心工具）

**需要完成的操作**：

- 注册GitHub或Gitee账号
- 在电脑上创建一个用于存放你的博客的文件夹（路径最好全英文，否则可能有意想不到的错误发生）



### 开始

---

下面是安装和创建博客的全步骤。



#### 配置Hexo

这一步将先创建网站的**基本框架**。

首先**安装Git，Node.js**。Windows在安装时注意勾选 "Add To Path"，以全局使用。

在安装完成后，打开命令行，分别输入git，node，若无报错且出现版本信息，即为安装成功。

创建一个专用于博客的文件夹，其路径名应是全英文。在该文件夹下打开命令行，输入如下命令。

```shell
npm install hexo-cli -g
# 全局安装hexo
hexo init blog
# 初始化创建blog文件夹
cd blog
# 移动到blog文件夹
npm install
# 初始化
```

在执行完所有命令后，博客的本地文件就已经搭建完成了。

![](https://pic.imgdb.cn/item/62ca684cf54cd3f937f15c69.png)

这时候，我们可以再输入以下命令以在本地**预览**我们的网站内容。

```shell
hexo server
# 或 hexo s
```

在输入后，按住Ctrl点击出现的链接即可跳转到浏览器。

![](https://pic.imgdb.cn/item/62ca6917f54cd3f937f22d8a.png)

如果出现这样的界面，那恭喜你，你已经做好自己的网站了，接下来只差修改内容和让别人看到他，预览完毕后可按 Ctrl + C 结束。



#### 配置本地Git

首先注册GitHub/Gitee账号，取决于你要使用哪个平台。

打开**Git bash**，输入如下命令。

```bash
git config --global user.name "你的GitHub/Gitee用户名"
git config --global user.email "你的GitHub/Gitee注册邮箱"
# 用于配置本地用户信息
ssh-keygen -t rsa -C "你的GitHub/Gitee注册邮箱"
# 用于生成SSH Key，之后出现三个选项，直接回车即可
```

由于你的本地 Git 仓库和 GitHub/Gitee 仓库之间的传输是通过**SSH加密**的，所以我们需要配置验证信息。Git通过**非对称加密的公钥与私钥**来完成加密，公钥放置在远程库（GitHub/Gitee）上，私钥放置在客户端上。每次同步都需要输入账号密码验证推送用户是否是合法用户，为了省去每次输入密码的步骤，在这里采用SSH。当进行推送时，Git会匹配公钥私钥是否合法，若匹配成功则允许推送。

我们的SSH公钥私钥文件默认存放在 **C:\Users\你的用户名\\.ssh\\** 路径下（Linux默认在 /home/.ssh 下）。



#### 配置GitHub/Gitee

##### Gitee 版

在登陆后，首先打开个人设置中的SSH公钥设置，打开我们在SSH公钥文件夹下的 **id_rsa.pub** 文件，将其内容全部复制，粘贴到公钥处。

![](https://pic.imgdb.cn/item/62ca7493f54cd3f937ffc93c.png)

然后在git bash 或 shell中输入如下代码判断是否成功：

![](https://pic.imgdb.cn/item/62ca78c3f54cd3f93705487e.png)

这样我们就可以使用Git直接同步仓库内容了。

之后在Gitee中创建一个仓库，仓库名称可以随意选取，但仓库路径**需要与你的用户名相同**，这样才可以用于创建网站。

此外，我们还需要添加Readme文件，其他不做要求。

<img src="https://pic.imgdb.cn/item/62ca713bf54cd3f937fbea03.png" style="zoom:80%;" />

在此步骤之后，复制仓库的SSH地址。

![](https://pic.imgdb.cn/item/62ca7a07f54cd3f937073e1c.png)



然后我们就可以打开 **blog** 文件夹了，打开里面的 **_config.yml** 文件，在其末尾修改编辑如下内容：

```yaml
deploy:
  type: git
  # 若之后出错，可用单引号括起git，即 'git'
  repo: git@gitee.com:flatig/blog.git
  # 这里是你刚刚复制的地址
  branch: main
  # 仓库分支，gitee一般为master，可在仓库首页查看
```

编辑完毕保存之后，即可进入下一步骤。



---



##### GitHub版

在登陆后，首先打开 Settings 中的 SSH and GPG keys，打开我们在SSH公钥文件夹下的 **id_rsa.pub** 文件，将其内容全部复制，粘贴到公钥处。

![](https://pic.imgdb.cn/item/62ca7728f54cd3f93702f8a5.png)



然后在git bash 或 shell中输入如下代码判断是否成功：

![](https://pic.imgdb.cn/item/62ca7882f54cd3f93704de9f.png)

这样我们就可以使用Git直接同步仓库内容了。

之后在GitHub中创建一个仓库，仓库名要求这样的格式： **用户名.github.io**

此外其他不做要求。在此步骤之后，复制仓库的SSH地址。

![](https://pic.imgdb.cn/item/62ca7a4bf54cd3f937079e63.png)

然后我们就可以打开 **blog** 文件夹了，打开里面的 **_config.yml** 文件，在其末尾修改编辑如下内容：

```yaml
deploy:
  type: git
  # 若之后出错，可用单引号括起git，即 'git'
  repo: git@github.com:Flatigers/Flatigers.github.io.git
  # 这里是你刚刚复制的地址
  branch: main
  # 仓库分支，GitHub一般为main，可在仓库首页查看
```

编辑完毕保存之后，即可进入下一步骤。



#### 同步到GitHub/Gitee

在完成上述所有操作后，在 blog 文件夹下打开终端或shell，输入如下命令：

```shell
npm install hexo-deployer-git --save
# 安装Hexo的git同步插件
```

在完成之后，输入如下命令，将仓库同步：

```shell
hexo clean
# 清除旧文件，可以简写为 hexo c
hexo generate
# 生成新文件，可以简写为 hexo g
hexo deploy
# 推送并同步，可以简写为 hexo d
# hexo新版本默认deploy前自动generate，故可省略hexo g

# 以上命令可以同时简写为：
# hexo c | hexo d

```

在同步完成之后，打开你的 GitHub/Gitee 仓库，你会看到多出了许多文件。



**使用GitHub的用户，此时已经可以访问到自己的网站了，只需要在浏览器输入 用户名.github.io**

使用Gitee的用户，还需要开启Pages服务。

![](https://pic.imgdb.cn/item/62ca802ef54cd3f937102504.png)



若出现问题，可以选择**强制启动HTTPS**，之后便可以通过提供的域名访问了。



### 完成

---

到这里已经基本完成了整个网站的搭建工作。

此外，还有一些别的操作，如绑**定高级域名，更换主题，设置评论区，自定义404**等，这些可以在**Hexo的官方文档**中找到，这里后续有可能会继续更新。

继续探索多彩的网络世界吧！









