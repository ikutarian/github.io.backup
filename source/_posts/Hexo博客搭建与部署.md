---
title: Hexo博客搭建与部署
date: 2018-09-27 10:03:24
tags:
  - Hexo
categories:
  - 安装与配置
---

之前一直在简书上写博客，但是后来简书要求除了绑定手机号以外，还需要绑定微信才让发文章，我就觉得很不爽。感觉博客还是自己维护比较自由，于是动了在 Github 利用 Hexo 写博客的心思

<!-- more -->

## 环境

* Git
* Node.js
* Hexo

## 安装 Git

进入[官网](https://git-scm.com/)，按照系统来选择安装即可

## 安装 Node.js

进入[官网](https://nodejs.org/en/)，按照系统来选择安装即可。只是注意一下要安装 LTS 版本

## 安装 Hexo

```
# 安装 Hexo
npm install hexo-cli -g

# 初始化一个名为 blog 的博客
hexo init blog

# 进入 blog 目录
cd blog

# 安装依赖
npm install
```

## Github 创建项目

创建一个仓库，名称为 `yourname.github.io`, 其中 `yourname` 是你的 Github 名称。比如我的名称是 `ikutarian`，于是我创建一个名为 `ikutarian.github.io` 的仓库

## 配置仓库地址

用编辑器打开你的 blog 项目，修改 `_config.yml` 文件。把创建好的仓库的地址复制进去

```
deploy:
  type: git
  repo: git@github.com:ikutarian/ikutarian.github.io.git
  branch: master
```

## 启动 Hexo 博客

```
hexo clean
hexo generate
hexo server
```

这样就可以在本地启动一个服务，然后在浏览器中输入 `http://localhost:4000` 就可以看见博客了

## 发布到 Github

```
hexo clean
hexo generate
hexo deploy
```

如果在执行 `hexo deploy` 时出现问题，需要安装一下 `npm install hexo-deployer-git --save`。以上步骤做好之后，过一会儿打开 `http://yourgithubname.github.io`，就可以看到博客了

## 主题配置

原生的主题不是很好看，我目前使用的是 Next 主题。认真看一遍主题的[使用文档](http://theme-next.iissnan.com/)，就会用了

### 字数统计

{% asset_img QQ20180929-174745@2x.png %}

像这样，可以显示字数和预估阅读时间

安装

```
npm i --save hexo-wordcount
```

然后修改 Next 主题的 _config.yml 配置

```
post_wordcount:
  item_text: true
  wordcount: true
  min2read: true
```

这样就行了

### 引用本地文件

如果想在 hexo 下加入一个文件，然后利用 URL 引用这个文件，要怎么做到呢？

在 `source\uploads` 下放入这个文件，然后使用 MarkDown 语法引用即可，比如

```
[本地下载](/uploads/VirtualXposed_0.16.1.apk)
```

### 文章引用

```
{% post_link 文章文件名 %}
```

比如我有一篇《{% post_link 理解Linux的硬链接和软链接%}》的文章，那我可以这么引用

```
{% post_link 理解Linux的硬链接和软链接 %}
```

## 如果换电脑了怎么办？

如果想再另外一台电脑上写博客，要怎么把这台电脑上的 Hexo 的项目文件迁移过去？可以利用 Git 的分支

### 创建新分支

当输入 `hexo deploy` 时，Hexo 会把生成好博客文件 push 到 `_config.yml` 文件配置好的仓库的地址。一般都是这么配置，都是在 master 分支上

```
deploy:
  type: git
  repo: git@github.com:ikutarian/ikutarian.github.io.git
  branch: master
```

于是，我们可以另外创建一个分支：hexo，用来存放 Hexo 的项目代码。步骤如下

```
git init
git checkout -b hexo
git add .
git commit -m "创建hexo分支，存放hexo的文件"
git remote add origin git@github.com:youname/youname.github.io.git
git push origin hexo
```

而且，不要忘了把主题文件也上传，我用的是 Next 主题。于是进入 themes/next 文件夹，执行

```
rm -rf .git
rm .gitignore
```

返回到主目录，删除 git 的缓存

```
git rm -r --cached .
git add .
git commit -m "把Next主题也上传"
git push origin hexo
```

### 在新电脑上要怎么操作？

先把代码拷贝下来

```
git clone git@github.com:ikutarian/ikutarian.github.io.git
```

切换到 hexo 分支

```
git checkout -b hexo origin/hexo
```

安装依赖

```
npm install
```

安装完 Hexo 会自动生成一个 helloword，删掉就行

```
rm source/_posts/hello-world.md 
```

以后写文章都在 hexo 分支下写，写完了本地运行看看

```
hexo clean; hexo generate; hexo server
```

没问题就可以部署了

```
hexo deploy
```

这样可以把生成好到文章 push 到 master 分支了。稍等一会儿，打开 youname.github.io 就能看到新文章了
