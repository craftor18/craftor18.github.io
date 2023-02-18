---
title: "关于如何在github上自动部署hugo博客的一些记录"
date: 2023-02-16T12:54:34+08:00
draft: false
categories : [
  "学习",
]
tags : [
  "hugo",
]
typora-root-url: ./
---

之前一直是在本地进行hugo博客推送，有时候真的手贱，然后，我的配置文件就全被我搞乱了，然后又没有备份，只能一个个重新修改，真的好烦，所以在网上找了好些教程，总算把我自己的博客源码放在了github并且自动部署hugo博客，下面是我自己的一些操作流程。

### 0 前置条件

默认已经安装了git、hugo和hugo的主题，并且如果有域名的话应该已经绑定了域名且使用了托管平台比如vercel托管平台。我的目录结构如下:

![blog_tree](/img/blog_tree.png)

### 1 博客源码连接github远程仓库

首先我们应该是要在本地建立hugo博客的，而且前置条件是默认我们已经安装好了git 和hugo以及hugo的主题，这里我安装的是使用比较广泛的even主题，至于怎么安装前面三个东西的教程我相信应该已经有许多教程了，我也是看了大佬的教程才配置好的，可以看看我前面的博客应该有放这些大佬的链接，下面我们看看如何将源码链接到远程仓库。

当然我们在此之前要在hugo博客源码的根目录下面先初始化git一下辣

`git init`

随后就可以链接远程仓库（ps：如果发现之前有其它的git分支，可以将整个目录所有的.git目录删掉，保证初始化正确执行，当然也会丢失之前所有git的缓冲区的内容哟）

`git remote add https://github.com/craftor18/craftor18.github.io.git  ## 链接远程仓库`

上面后面的仓库地址更改为自己的仓库即可

```shell
## 本地创建名为hugo的新分支
git branch hugo
## 切换到这个新的hugo分支
git checkout hugo

##当然我们也可以在创建hugo分支的同时直接切换到hugo分支
git checkout -b hugo

## 添加当前源码目录下所有的内容到暂存区
git add .
## 将暂存区的内容放到本地仓库并且添加提交信息
git commit -m "first code commit"
## 将本地仓库的内容推送到远程仓库，这里就是将整个博客源码目录全部提交到了github上，且分支为hugo，如果github远程仓库上原本就有hugo分支则会显示无法推送哟，需要先pull一下
git push origin hugo
```



### 2 github远程部署

为了能将github上hugo分支的public文件夹能够部署到另外一个网页gh-pages分支上，我们首先要在github上绑定本地创建的公钥和私钥。

```shell
## git本地创建公钥和私钥
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
## 随后我们可以cat打开查看里面两个文件的内容。.pub的是公钥
```

我们将公钥和私钥的内容分别添加到github上我们自己创建的仓库里，打开仓库的settings就可以设置

![githubset](/img/githubset.png)

然后在里面找到deploy keys和codespaces，将公钥和私钥分别绑定，名称都为`ACTIONS_DEPLOY_KEY`。

![keys](/img/key.png)

然后我们打开仓库的Actions页面，向里面添加一个新的模板workflow并且将内容修改为如下：

```yaml
name: github pages

on:
  push:
    branches:
      - hugo  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: false  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.79.1'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }} ##可以不用，如果有上面那个秘钥的话
          publish_dir: ./public
          cname: craftor18.top  ##这里可以更换为自己的域名
          publish_branch: gh-pages
```

上面的工作流主要是基于博客源码和网页生成目录在同一个仓库下的，如果想将博客源码存于另一个仓库的话，还需要加上别的参数，我这里不太懂，因为之前失败了，可以参考一些别人的教程。哈哈哈



### 3 总结

总之就是在已经本地安装了hugo博客和已经有域名的情况下，如何将源码和网页同时置于一个仓库管理的流程辣，其实博客的源码还是放在一个私有的仓库会好一些，不过我反正也无所谓因此就放到了一起。下面是我参考别人大佬的一些教程链接。

[使用 Github Actions 自动部署 Hugo 博客 | ZenLian](https://zenlian.github.io/posts/tools/github-actions-hugo/)

[从零开始搭建个人博客网站系列 三、hugo站点自动部署到vercel托管平台 - 码农在新加坡的个人博客 (leftpocket.cn)](https://www.leftpocket.cn/post/hugo/hugo_vercel/)
