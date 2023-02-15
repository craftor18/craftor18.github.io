---
title: "浅浅记录一下hugo的简易部署流程（阿里域名版）"
date: 2023-02-07T14:44:16+08:00
categories : [
"学习",
]
tags: [
	"hugo",
]
typora-root-url: ./
---

折腾了许久，总算将一个勉强能看的hugo博客页面搞了出来，本意还是为了记录一下自己的学习笔记，其实我对于前端的知识不太熟，只了解一点点html和css的基础，JavaScript完全不懂。因为我是在Windows下搞的，因此流程大致如下。

## 1 安装git

到git的官网下载windows版本的git，跟着说明一步步安装就行了。

[Git - Downloading Package (git-scm.com)](https://git-scm.com/download/win)

为了后面能在git上快速安装hugo，要先安装go依赖，去官网下载默认安装即可。

https://go.dev/doc/install

go编译安装hugo的过程需要用到gcc，因此要去下载安装一个minGW。

[MinGW-w64 - for 32 and 64 bit Windows download | SourceForge.net](https://sourceforge.net/projects/mingw-w64/)

## 2 安装hugo

前面所有软件装完之后就可以安装hugo了，这个很简单，官网也介绍了，打开git bash敲入一行代码即可解决。

`go install -tags extended github.com/gohugoio/hugo@latest`

等待完成后在git bash页面敲`hugo version`看看hugo是否安装成功。

## 3 使用hugo开始搭建自己的博客

找一个文件夹，在git命令行下cd进入，然后输入：

`hugo new site yoursitename`

搞完后去找一个主题git clone一下到themes目录下:

`git clone https://github.com/olOwOlo/hugo-theme-even themes/even`

然后去even目录下的exampleSite目录找到config.toml文件将里面的内容复制到网站根目录下的config.toml里覆盖掉原来的内容，这样就得到一个标准的even博客页面辣。

**发布一篇博客**

`hugo new post/first_post.md`

然后就可以在这个first_post.md文件里写自己的笔记辣

**本地检视博客内容**

`hugo server`

ps:图片的根目录是 `static` 目录哟，也就是你在`static`目录创建文件夹并且存放文件，但是在有even主题的情况下，这个`static`目录是指`even`目录下的`static`目录而不是根目录下的`static`目录。

## 4 将博客部署到github上并且利用vercel绑定域名托管

这个怎么部署到github和托管vercel我就不详细说了，建议看大佬的教程：

[从零开始搭建个人博客网站系列 三、hugo站点自动部署到vercel托管平台 - 码农在新加坡的个人博客 (leftpocket.cn)](https://www.leftpocket.cn/post/hugo/hugo_vercel/)

[从零开始搭建个人博客网站系列 四、申请免费域名并绑定到个人博客 - 码农在新加坡的个人博客 (leftpocket.cn)](https://www.leftpocket.cn/post/hugo/hugo_dns/)

我就说一下我这里与大佬的那个申请域名和dns解析的不同，因为我用的是阿里云的域名，所以要在阿里云上购买，买那个一年9块钱的域名（购买需实名认证哟），买完可以看到

![dnsyuming](/img/yuming.png)

点击解析，到了解析界面就可以发现与那个腾讯的dns解析服务有类似的操作，然后照着添加A记录和CNAME记录即可，最后vercel里面就会自动部署了，每一次更新博客都可以在本地git bash下面push掉hugo生成的public文件夹，这样远程也会自动部署更新网页。（ps：可以本地搞几个软连接，然后就不用每次来回复制文件了，这个教程也可看那个大佬滴哈哈哈）



总之，我这个真的是浅浅记录一下，因为好多大佬都写过类似流程早已完善了，因此我只是对不同之处作个介绍，如有不当和过于菜鸟水平之处，还请各位大佬多多指点啦。
